pipeline {
    agent { label 'docker' }

    parameters {
        string(
            name: 'HOST',
            defaultValue: '',
            description: 'Target hostname or IP address'
        )
        credentials(
            name: 'SSH_CREDENTIALS',
            description: 'SSH username and password for the target host',
            credentialType: 'com.cloudbees.plugins.credentials.impl.UsernamePasswordCredentialsImpl',
            required: true
        )
    }

    stages {
        stage('Update packages') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: params.SSH_CREDENTIALS,
                    usernameVariable: 'SSH_USER',
                    passwordVariable: 'SSH_PASS'
                )]) {
                    // SSHPASS env var keeps the password out of CLI args; heredoc expands it locally for sudo -S
                    sh '''
                        export SSHPASS="$SSH_PASS"
                        sshpass -e ssh \
                            -o StrictHostKeyChecking=no \
                            -o BatchMode=no \
                            -o ConnectTimeout=30 \
                            "$SSH_USER@$HOST" bash << EOF
printf '%s\n' "$SSHPASS" | sudo -S apt-get update &&
printf '%s\n' "$SSHPASS" | sudo -S env DEBIAN_FRONTEND=noninteractive apt-get upgrade -y &&
printf '%s\n' "$SSHPASS" | sudo -S apt-get autoremove -y &&
printf '%s\n' "$SSHPASS" | sudo -S apt-get clean
EOF
                    '''
                }
            }
        }

        stage('Reboot if required') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: params.SSH_CREDENTIALS,
                    usernameVariable: 'SSH_USER',
                    passwordVariable: 'SSH_PASS'
                )]) {
                    // reboot itself needs elevation; the file check does not
                    sh '''
                        export SSHPASS="$SSH_PASS"
                        sshpass -e ssh \
                            -o StrictHostKeyChecking=no \
                            -o ConnectTimeout=30 \
                            "$SSH_USER@$HOST" bash << EOF || true
if [ -f /var/run/reboot-required ]; then
    echo "Reboot required. Rebooting..."
    printf '%s\n' "$SSHPASS" | sudo -S reboot now
else
    echo "No reboot required."
fi
EOF
                    '''
                }
            }
        }
    }
}

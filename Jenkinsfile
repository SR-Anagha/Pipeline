pipeline {
    agent any

    tools {
        maven 'MAVEN_HOME'
    }

    environment {
        // GitHub Maven archetype repository URL
        SOURCE_URL = 'https://github.com/apache/maven-archetypes/archive/refs/heads/master.zip'
        SOURCE_DIR = 'maven-archetypes-master'  // Directory where the source code will be extracted
        JAR_FILE = 'target/myapp.jar'  // The path to the generated JAR file
        ANSIBLE_INVENTORY = 'inventory'  // Path to Ansible inventory file
        ANSIBLE_PLAYBOOK = 'deploy.yml'  // Path to Ansible playbook
    }

    stages {
        stage('Download Source Code') {
            steps {
                script {
                    // Download the source code from GitHub (ZIP format)
                    echo 'Downloading source code from GitHub...'
                    sh "wget ${SOURCE_URL} -O /tmp/source.zip"
                    
                    // Extract the downloaded source code
                    echo 'Extracting source code...'
                    sh 'unzip /tmp/source.zip -d /tmp'
                    
                    // Rename extracted directory to the expected name
                    sh "mv /tmp/maven-archetypes-master ${SOURCE_DIR}"
                }
            }
        }

        stage('Build') {
            steps {
                dir(SOURCE_DIR) {
                    // Build the Maven project
                    echo 'Building Maven project...'
                    sh 'mvn clean package'
                }
            }
        }

        stage('Test') {
            steps {
                dir(SOURCE_DIR) {
                    // Run Maven tests
                    echo 'Running Maven tests...'
                    sh 'mvn test'
                }
            }
        }

        stage('Archive Artifacts') {
            steps {
                dir(SOURCE_DIR) {
                    // Archive the build artifacts (JAR files)
                    echo 'Archiving build artifacts...'
                    archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    // Run Ansible playbook for deployment
                    echo 'Deploying application using Ansible...'
                    
                    // Define the copy and restart tasks for Ansible
                    def deployTasks = '''
                    - name: Deploy Maven Application
                      hosts: web
                      tasks:
                        - name: Copy JAR file to remote server
                          copy:
                            src: ${WORKSPACE}/${SOURCE_DIR}/${JAR_FILE}
                            dest: /opt/myapp.jar
                            mode: '0755'

                        - name: Restart Application Service
                          shell: |
                            pkill -f myapp.jar || true
                            nohup java -jar /opt/myapp.jar > /dev/null 2>&1 &
                    '''
                    
                    // Write the tasks to a temporary file for Ansible execution
                    writeFile file: '/tmp/deploy.yml', text: deployTasks
                    
                    // Run the Ansible playbook
                    sh "ansible-playbook -i ${ANSIBLE_INVENTORY} /tmp/deploy.yml"
                }
            }
        }
    }
}

pipeline {
    agent any

    options {
        // Skip default SCM checkout; we'll handle it manually
        skipDefaultCheckout(true)
    }

    stages {
        stage('Cleanup and Checkout') {
            steps {
                echo ">>> Cleaning workspace and checking out repository..."
                cleanWs()
                git url: 'https://github.com/horizoninfinite/apacheoncicd.git', branch: 'main'
                echo ">>> Repository cloned successfully."
                sh 'ls -la'
            }
        }

        stage('Update Packages') {
            steps {
                echo ">>> Updating system packages using dnf..."
                sh '''
                    sudo dnf update -y
                '''
            }
        }

        stage('Install Apache') {
            steps {
                echo ">>> Installing Apache (httpd)..."
                sh '''
                    sudo dnf install -y httpd
                '''
            }
        }

        stage('Deploy Custom Index Page from Repo') {
            steps {
                echo ">>> Deploying custom index.html to Apache document root..."
                sh '''
                    # Define paths
                    APACHE_DOCUMENT_ROOT="/var/www/html"
                    REPO_INDEX_PATH="index.html"

                    if [ -f "$REPO_INDEX_PATH" ]; then
                        echo "Found index.html in repo. Overwriting existing file..."
                        # Force copy and set correct permissions
                        sudo cp -f "$REPO_INDEX_PATH" "$APACHE_DOCUMENT_ROOT/index.html"
                        sudo chown apache:apache "$APACHE_DOCUMENT_ROOT/index.html"
                        sudo chmod 644 "$APACHE_DOCUMENT_ROOT/index.html"
                        echo "index.html successfully deployed to $APACHE_DOCUMENT_ROOT/"
                    else
                        echo "ERROR: index.html not found in repo."
                        exit 1
                    fi
                '''
            }
        }

        stage('Start and Enable Apache Service') {
            steps {
                echo ">>> Starting and enabling Apache (httpd) service..."
                sh '''
                    sudo systemctl start httpd
                    sudo systemctl enable httpd
                    echo "Apache service is running and enabled on boot."
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                echo ">>> Verifying that Apache is serving the index page..."
                sh '''
                    # Check if Apache is serving the index page on localhost
                    STATUS_CODE=$(curl -s -o /dev/null -w "%{http_code}" http://localhost)

                    if [ "$STATUS_CODE" = "200" ]; then
                        echo "SUCCESS: Apache is serving the custom index.html."
                    else
                        echo "WARNING: Apache did not return HTTP 200. Status code: $STATUS_CODE"
                        exit 1
                    fi
                '''
            }
        }
    }
}

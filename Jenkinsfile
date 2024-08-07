pipeline {       
    agent any 

    environment {
        AWS_ACCESS_KEY_ID = credentials('aws-access-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-access-key')
        AWS_REGION = 'ap-south-1'
        CLUSTER_NAME = 'kong-EKSClusterRole'
        HELM_BIN = '/usr/local/bin/helm'
        SERVICE_ACCOUNT = 'jenkins-sa'
        NAMESPACE_CP = 'kong-cp-kong'
        NAMESPACE_DP = 'kong-dp-kong'
        CERTS_DIR = './certs' // Path to your certificate files
        LICENSE_FILE = 'license.json' // Path to your Kong license file
    }

    stages {
        stage('Configure AWS CLI') {
            steps {
                withCredentials([string(credentialsId: 'aws-access-key-id', variable: 'AWS_ACCESS_KEY_ID'),
                                 string(credentialsId: 'aws-secret-access-key', variable: 'AWS_SECRET_ACCESS_KEY')]) {
                    sh '''
                    aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
                    aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
                    aws configure set region $AWS_REGION
                    '''
                }
            }
        }

        stage('Update kubeconfig') {
            steps {
                sh '''
                aws eks update-kubeconfig --name $CLUSTER_NAME --region $AWS_REGION
                '''
            }
        }

        stage('Setup Namespaces') {
            steps {
                script {
                    sh 'kubectl create namespace $NAMESPACE_CP || true'
                    sh 'kubectl create namespace $NAMESPACE_DP || true'
                }
            }
        }

        stage('Create Secrets') {
            steps {
                script {
                    sh '''
                        kubectl create secret tls kong-cluster-cert --cert=${CERTS_DIR}/tls.crt --key=${CERTS_DIR}/tls.key -n $NAMESPACE_CP || true
                        kubectl create secret tls kong-cluster-cert --cert=${CERTS_DIR}/tls.crt --key=${CERTS_DIR}/tls.key -n $NAMESPACE_DP || true
                        
                        kubectl create secret generic postgresql-password --from-literal=postgres-password=kong --from-literal=password=kong -n $NAMESPACE_CP || true
                        
                        kubectl create secret generic kong-enterprise-license --from-file=license=${LICENSE_FILE} -n $NAMESPACE_CP || true
                        kubectl create secret generic kong-enterprise-license --from-file=license=${LICENSE_FILE} -n $NAMESPACE_DP || true
                        
                        kubectl create secret generic kong-enterprise-superuser-password --from-literal=password=password -n $NAMESPACE_CP || true
                        
                        x='{"cookie_name":"admin_session","storage":"kong","cookie_samesite":"off","cookie_secure":false,"secret":"secret"}'
                        kubectl create secret generic kong-session-config --from-literal=admin_gui_session_conf="$x" -n $NAMESPACE_CP || true
                    '''
                }
            }
        }

        stage('Deploy Control Plane') {
            steps {
                script {
                    sh '''
                        helm repo add kong https://charts.konghq.com
                        helm repo update
                        helm upgrade --install -f ./kong_values/kong_cp_values.yaml kong-cp kong/kong --namespace $NAMESPACE_CP
                    '''
                }
            }
        }

        stage('Deploy Data Plane') {
            steps {
                script {
                    def cluster_ep = sh(script: "kubectl get svc --namespace $NAMESPACE_CP kong-cp-kong-kong-cluster -o jsonpath='{.status.loadBalancer.ingress[0].ip}:8005'", returnStdout: true).trim()
                    def telem_ep = sh(script: "kubectl get svc --namespace $NAMESPACE_CP kong-cp-kong-kong-clustertelemetry -o jsonpath='{.status.loadBalancer.ingress[0].ip}:8006'", returnStdout: true).trim()
                    
                    sh """
                        helm upgrade --install -f ./kong_values/kong_dp_values.yaml kong-dp --set env.cluster_control_plane=${cluster_ep} --set env.cluster_telemetry_endpoint=${telem_ep} kong/kong --namespace $NAMESPACE_DP
                    """
                }
            }
        }
    }
}









// pipeline {      
//     agent any 

//     environment {
//         AWS_ACCESS_KEY_ID = credentials('aws-access-key-id')
//         AWS_SECRET_ACCESS_KEY = credentials('aws-secret-access-key')
//         AWS_REGION = 'ap-south-1'
//         CLUSTER_NAME = 'kong-EKSClusterRole'
//         HELM_BIN = '/usr/local/bin/helm'
//         SERVICE_ACCOUNT = 'jenkins-sa'
//         NAMESPACE_CP = 'kong-cp-kong'
//         NAMESPACE_DP = 'kong-dp-kong'
//         CERTS_DIR = './certs' // Path to your certificate files
//         LICENSE_FILE = 'license.json' // Path to your Kong license file
//     }

//     stages {
//         stage('Configure AWS CLI') {
//             steps {
//                 withCredentials([string(credentialsId: 'aws-access-key-id', variable: 'AWS_ACCESS_KEY_ID'),
//                                  string(credentialsId: 'aws-secret-access-key', variable: 'AWS_SECRET_ACCESS_KEY')]) {
//                     sh '''
//                     aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
//                     aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
//                     aws configure set region $AWS_REGION
//                     '''
//                 }
//             }
//         }

//         stage('Update kubeconfig') {
//             steps {
//                 sh '''
//                 aws eks update-kubeconfig --name $CLUSTER_NAME --region $AWS_REGION
//                 '''
//             }
//         }

//         stage('Setup Namespaces') {
//             steps {
//                 script {
//                     sh 'kubectl create namespace $NAMESPACE_CP || true'
//                     sh 'kubectl create namespace $NAMESPACE_DP || true'
//                 }
//             }
//         }

//         stage('Create Secrets') {
//             steps {
//                 script {
//                     sh '''
//                         kubectl create secret tls kong-cluster-cert --cert=${CERTS_DIR}/tls.crt --key=${CERTS_DIR}/tls.key -n $NAMESPACE_CP || true
//                         kubectl create secret tls kong-cluster-cert --cert=${CERTS_DIR}/tls.crt --key=${CERTS_DIR}/tls.key -n $NAMESPACE_DP || true
                        
//                         kubectl create secret generic postgresql-password --from-literal=postgres-password=kong --from-literal=password=kong -n $NAMESPACE_CP || true
                        
//                         kubectl create secret generic kong-enterprise-license --from-file=license=${LICENSE_FILE} -n $NAMESPACE_CP || true
//                         kubectl create secret generic kong-enterprise-license --from-file=license=${LICENSE_FILE} -n $NAMESPACE_DP || true
                        
//                         kubectl create secret generic kong-enterprise-superuser-password --from-literal=password=password -n $NAMESPACE_CP || true
                        
//                         x='{"cookie_name":"admin_session","storage":"kong","cookie_samesite":"off","cookie_secure":false,"secret":"secret"}'
//                         kubectl create secret generic kong-session-config --from-literal=admin_gui_session_conf="$x" -n $NAMESPACE_CP || true
//                     '''
//                 }
//             }
//         }

//         stage('Deploy Control Plane') {
//             steps {
//                 script {
//                     sh '''
//                         helm repo add kong https://charts.konghq.com
//                         helm repo update
//                         helm install -f ./kong_values/kong_cp_values.yaml kong-cp kong/kong --namespace $NAMESPACE_CP
//                     '''
//                 }
//             }
//         }

//         stage('Deploy Data Plane') {
//             steps {
//                 script {
//                     def cluster_ep = "kong-cp-kong-cluster.$NAMESPACE_CP.svc.cluster.local:8005"
//                     def telem_ep = "kong-cp-kong-clustertelemetry.$NAMESPACE_CP.svc.cluster.local:8006"
                    
//                     sh """
//                         helm install -f ./kong_values/kong_dp_values.yaml kong-dp --set env.cluster_control_plane=${cluster_ep} --set env.cluster_telemetry_endpoint=${telem_ep} kong/kong --namespace $NAMESPACE_DP
//                     """
//                 }
//             }
//         }
//     }
// }

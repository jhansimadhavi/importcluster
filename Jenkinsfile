#!/usr/bin/env groovy

pipeline {
  agent any
  environment {
      CLUSTER_NAME="import2"
      BLUEPRINT="default"
      PROJECT="jhansi"
      KUBECONFIG="/tmp/import2_kubeconfig"
      REGION="us-west-2"
  }
  stages {
    stage("Checkout Repo") {
      steps {
            checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/RafaySystems/rafay-cicd-helpers']]])

      }
    }
    stage("Cluster Import") {
        steps {
            sh label: '', script: '''
            #!/bin/bash
            rm -f rctl-linux-amd64.tar.bz2
            wget -q https://s3-us-west-2.amazonaws.com/rafay-prod-cli/publish/rctl-linux-amd64.tar.bz2
            tar -xf rctl-linux-amd64.tar.bz2
            chmod 0755 rctl
            ./rctl create cluster imported ${CLUSTER_NAME} -l aws/${REGION} -b ${BLUEPRINT} -p ${PROJECT} --config=/tmp/rctlconf> cluster_bootstrap.yaml
            grep -i "cluster.rafay.dev" cluster_bootstrap.yaml > /dev/null 2>&1
            if [ $? -eq 0 ];
            then
              set +e
              kubectl get ns|grep rafay-system > /dev/null 2>&1
              if [ $? -eq 1 ];
              then
                set -e
                echo "namespace rafay-system not found.... applying bootstrap"
                sleep 30
                kubectl apply -f cluster_bootstrap.yaml
              else
                echo "namespace rafay-system found... cluster is already imported"
                exit 1
              fi
            fi
            '''
        }
    }
  }
  }

# aws-ecs

# Crear Instancia EC2 con Auto-Scaling y Reglas de Seguridad

Este conjunto de comandos de AWS CLI te permite crear una instancia EC2 con auto-escalado y definir reglas de seguridad para tu aplicación.

## Pasos:

### 1. Crear una Instancia EC2

```bash
aws ec2 run-instances \
    --image-id ami-xxxxxxxxxxxxx \  
    --instance-type t2.micro \       
    --key-name my-key-pair \         
    --security-groups MySecurityGroup \ 
    --user-data '#!/bin/bash
                 # Instalar Node.js
                 curl -sL https://deb.nodesource.com/setup_14.x | sudo -E bash -
                 sudo apt-get install -y nodejs'
```

### 2. Crear una Configuración de Lanzamiento

```bash
aws autoscaling create-launch-configuration \
    --launch-configuration-name MyLaunchConfig \
    --image-id ami-xxxxxxxxxxxxx \
    --instance-type t2.micro \
    --key-name my-key-pair \
    --security-groups MySecurityGroup \
    --user-data '#!/bin/bash
                 curl -sL https://deb.nodesource.com/setup_14.x | sudo -E bash -
                 sudo apt-get install -y nodejs'
```

### 3. Crear un Grupo de Auto-Escalado

```bash
aws autoscaling create-auto-scaling-group \
    --auto-scaling-group-name MyAutoScalingGroup \
    --launch-configuration-name MyLaunchConfig \
    --min-size 1 \
    --max-size 3 \
    --desired-capacity 2 \
    --availability-zones us-east-1a us-east-1b \ 
    --vpc-zone-identifier subnet-xxxxxxxxxxxxx \ 
    --tags Key=Name,Value=MyAutoScalingGroup,PropagateAtLaunch=true
```

### 4. Establecer Política de Escalado hacia Arriba

```bash
aws autoscaling put-scaling-policy \
    --auto-scaling-group-name MyAutoScalingGroup \
    --policy-name ScaleUpPolicy \
    --scaling-adjustment 1 \
    --adjustment-type ChangeInCapacity \
    --cooldown 300 \
    --metric-name CPUUtilization \ 
    --namespace AWS/EC2 \
    --statistic Average \
    --comparison-operator GreaterThanOrEqualToThreshold \
    --threshold 70 \
    --evaluation-periods 2
```

### 5. Establecer Política de Escalado hacia Abajo

```bash
aws autoscaling put-scaling-policy \
    --auto-scaling-group-name MyAutoScalingGroup \
    --policy-name ScaleDownPolicy \
    --scaling-adjustment -1 \
    --adjustment-type ChangeInCapacity \
    --cooldown 300 \
    --metric-name CPUUtilization \
    --namespace AWS/EC2 \
    --statistic Average \
    --comparison-operator LessThanOrEqualToThreshold \
    --threshold 30 \
    --evaluation-periods 2
```

### 6. Autorizar Tráfico Entrante

```bash
aws ec2 authorize-security-group-ingress \
    --group-name MySecurityGroup \
    --protocol tcp \
    --port 22 \ 
    --source 0.0.0.0/0 \
    --region us-east-1 \
```



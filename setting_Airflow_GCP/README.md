Paso a paso, actividad: Crear instancia de Airflow en GCP con Terraform

1. Al tener OS Windows, se debe instalar ubuntu a través del wsl, para lo cual se corre el siguiente comando desde el PowerShell:
	wsl --install
Después de un buen rato, se descargan todos la información necesaria, al final solicita el reinicio de la compu, al hacerlo el sistema automáticamente instala Ubuntu (si no es especifica alguna otra distribución de Linux), después de todo ya puedes correr wsl y trabajar la práctica sin problemas.

*Cuando se trabaja con WSL, hay que recordar que para cambiar entre directorios se debe usar la estructura cd /mnt/c/users... Todos los comandos de Linux se corren desde el floder mnt.

2. Después de instalar todo, se trae el repositorio base con el que se va a trabajar la práctica alojado en: https://github.com/wizelineacademy/data-bootcamp-terraforms

3. Teniendo los archivos base para poder hacer la práctica, procedemos a instalar el siguiente software:
	-Terraform: Se corren los siguientes comandos
		sudo apt-get update && sudo apt-get install -y gnupg software-properties-common curl (se instala y actualiza las funciones de apt -instalador de paquetes-)
		curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add - (añade la API key del proyecto Terraform al entorno Linux)
		sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main" (agrega el repositorio de hashicorp -Terraform- al entorno de Linux)
		sudo apt-get update && sudo apt-get install terraform (Finalmente con este comando se instala terraform)
	 Se puede verificar la instalación de terraform aplicando alguno de los siguientes comandos: terraform version / terraform -help plan

	-Google Cloud SDK: Se corren los siguientes comandos
		echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] https://packages.cloud.google.com/apt cloud-sdk main" | sudo tee -a /etc/apt/sources.list.d/google-cloud-sdk.list (de manera análoga que en Terraform, se utiliza este comando para agregar la URL de GCP en entorno Linux)
		sudo apt-get install apt-transport-https ca-certificates gnupg (se instala o se verifica que esté instalado el APT-TRANSPORT-HTTPS, el cuál se encarga de instalar paquetes con certiicados firmados HTTPs)
		curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key --keyring /usr/share/keyrings/cloud.google.gpg add - (importa la llave pública de la API de GCP al entorno local)
		sudo apt-get update && sudo apt-get install google-cloud-sdk (Con este comando finalmente se instala GCP en nuestra máquina)
	-Kubernetes / Kubectl: Es el paquete de contenedores con el cual se pueden instanciar los clústeres en GCP
		curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl" (Realiza la instalación a través de CRUL)
		sudo chmod +x ./kubectl (habilita los permisos de ejecución del binario kubectl)
		sudo mv ./kubectl /usr/local/bin/kubectl (Mueve y/o agrega el binario de kubectl al path para no tener lío al llamarlo)
		kubectl version --client (Verifica la instalación de Kubernetes)
	-Helm: Un administrador de paquetes para kubernetes, se corren los siguientes comandos
		curl https://baltocdn.com/helm/signing.asc | sudo apt-key add -
		sudo apt-get install apt-transport-https --yes
		echo "deb https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
		sudo apt-get update
		sudo apt-get install helm
4. Se procede con la práctica como tal, para ello se inicializa terraform con el comando:
	sudo terraform init
5. Se "inicializan" los servicios de Google Cloud con el comando:
	gcloud init
y se selecciona el proyecto en el que se está trabajando
6. Se habilita a google cloud platform para que autorice utilizar todos los proyectos de nuestra cuenta de Google con el comando:
	gcloud auth application-default login
7. Se aplican las variables del archivo de terraform terraform.tvars, al finalizar este proceso habremos configurado nuestro primer cluster para poder usar Airflow en la nube con Kubernetes.
8. Para poder instanciar y utilizar Airflow desde la nube debemos indicarle a gcp que utilice o "desempaquete" los containers que cargamos a través de Terraform, para ello corremos el comando
	gcloud container clusters get-credentials $(terraform output -raw kubernetes_cluster_name) --region $(terraform output -raw location)
9. Con Kubernetes creamos el namespace "airflow" que va a ser el "alias" de nuestro Airflow, el cuál va a entender Kubernetes con el siguiente comando:
	kubectl create namespace airflow
10. Con ayuda de Helm, le indicamos que en nuestro espacio "airlfow" creado en el paso anterior va a trabajar con los paquetes de Airflow, que están en su repositorio en su pagina web, traemos el repositorio de Ariflow de la página de Apache, para ello corremos el código:
	helm repo add apache-airflow https://airflow.apache.org
11. Del repo creado instalamos Airflow en nuestro cluster de GCP, para lo cual corremos el comando:
	helm install airflow apache-airflow/airflow --namespace airflow
*Si por alguna razón genera algun error y no queda instalado Airflow, se debe volver a correr el comando con de la siguiente manera
	helm install --replace airflow apache-airflow/airflow --namespace airflow
Al poner el --replace obligamos que sobreescriba cualquier cosa que se haya hecho anteriormente

Cuando Helm instala Airflow nos entrega el username y el password tanto del Login de nuestra UI como de las BD de postgres

12. Verificamos que los nodos estén corriendo correctamente en GCP con el comando
	kubectl get pods -n airflow
13. Ahora, para poder empezar a usar Airflow de una manera local aprovechando los recursos de la nube corremos el comando
	kubectl port-forward svc/airflow-webserver 8080:8080 --namespace airflow
El cuál hace un port forwarding desde nuestra nube GCP hacia nuestro localhost en el puerto 8080
14. Habiendo hecho esto, podemos acceder a Airflow en nuestro navegador a través de la siguiente URL
	localhost:8080
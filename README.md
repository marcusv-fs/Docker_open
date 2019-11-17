# Docker_open
Repositório contendo um tutorial e o código de uma aplicação raspberry para abrir uma porta rodando em um container Docker.

# O que é Docker?

Docker é uma plataforma que facilita a criação e administração de ambientes isolados. Ele funciona a partir do empacotamento de um ambiente inteiro ou de uma aplicação para dentro de um container, o que o torna portável para qualquer outro host que contenha o Docker Instalado. 
Ou seja, é criada certa independência do sistema operacional do usuário, pois o ambiente necessário para rodar a aplicação já está previamente configurado e será reproduzido junto do programa, devido a isso, o tempo gasto para disponibilizar a aplicação é reduzida juntamente com os problemas de incompatibilidade. 
Uma outra vantagem desse sistema é o fato de ela utilizar como backend default o LXC, o que permite que se limite o uso de recursos por container (CPU, I/O, Memória e outros).
Além disso, deve-se ressaltar, ainda, que o Docker funciona de maneira distinta de uma máquina virtual comum, pois ela não necessita do Hypervisor, que desperdiça muitos recursos por simular todo um sistema operacional. Ele se utiliza das features do próprio sistema operacional hospedeiro para gerar os containers, isso o torna capaz de inicializar mais rápido e ser mais eficiente no uso de seus recursos.

# Criando um container que abre uma porta usando GPIO do Raspberry

### Passo 1: Instalar o Docker no Raspberry
A primeiro passo a se tomar é instalar o raspbian no seu Raspberry, e, posteriormente, verificar por atualizações. O site para o download do raspbian é: https://www.raspberrypi.org/downloads/raspbian/.
Com o Raspberry pronto, verifique por atualizações usando o seguinte comando:
```
sudo apt-get update && sudo apt-get upgrade
```
Em seguida instale o Docker:
```
curl -sSL https://get.docker.com | sh
```
Agora, adicione ao usuário as permissões para rodar os comandos do Docker. Para isso digite: 
```
sudo usermod -aG docker pi
```
Por fim, verifique a instalação do Docker usando o comando:
```
docker –version
```
### Passo 2: Criando um diretório
Para manter os arquivos em ordem é recomendável criar um diretório específico para a imagem Docker que iremos criar. Nesse tutorial o nomearemos “docker_open”:
```
mkdir docker_open
cd docker_open
```

### Passo 3: Criando um script em Python
Nesse passo criaremos um script em Python para acionar a relé que abrirá a porta, entretanto, caso se deseje, é possível usar outro script no lugar. Crie ele no diretório que acabamos de criar, nesse exemplo ele se chamará “abrir.py”. 
```
import RPi.GPIO as GPIO
import time
GPIO.setmode(GPIO.BOARD)
GPIO.setup(8, GPIO.OUT)
GPIO.output(8, GPIO.LOW)
time.sleep(5)
GPIO.cleanup()
print("Sucesso")
```

### Passo 4: Criando o Dockerfile
Agora é hora de criar a imagem que será usada pelo Docker para a construção do Container. Para isso crie um arquivo chamado “Dockerfile” e dentro dele insira o seguinte código:
```
FROM raspbian/stretch
ADD abrir.py ./
RUN apt-get update -y && apt-get install python-pip -y && pip install rpi.gpio
CMD [“python”, “./abrir.py”]
```

### Passo 5: Criando a imagem Docker a partir do Dockerfile
Agora criaremos a imagem Docker usando o Dockerfile que acabamos de criar, nomearemos a imagem como “Teste” e vamos marcar sua versão como “1.0”:
```
docker build -t "teste:1.0" .
```
Depois de executado o comando, o Docker vai construir a imagem. Esse processo pode demorar alguns minutos caso seja a primeira execução. Quando terminada a ação, teste se a imagem se encontra na lista do Docker:
```
Docker image ls
```
### Passo 6: Prepare o circuito do Raspberry
Agora conecte os cabos do relé nas posições corretas do Raspberry. No código usamos a entrada GPIO 8 como receptora de sinal. A alimentação é no 5v e o GND no GND.

### Passo 7: Inicie o container
Ao fim desse processo basta iniciar o container, entretanto, por se tratar de um container que vai operar a GPIO, é necessário passar como parâmetro “--privileged”. Para executar o container usando a imagem que acabamos de criar basta usar o comando:
```
Docker run --privileged teste:1.0
```

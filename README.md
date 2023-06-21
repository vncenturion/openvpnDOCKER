# openvpnDOCKER

Configuração de vpn-server em docker utilizando imagem docker: [kylemanna/openvpn](https://hub.docker.com/r/kylemanna/openvpn/).

A imagem kylemanna/openvpn inclui scripts para gerar automaticamente:

* Parâmetros Diffie-Hellman
* Uma chave privada
* Um certificado autoassinado correspondente à chave privada para o servidor OpenVPN
* Uma chave e certificado EasyRSA CA
* Uma chave de autenticação TLS a partir da segurança HMAC

## Pré-requisitos

Para a prática serão utilizadas 3 máquinas virtuais ubuntu 22.04.2 LTS (memoria 2Gb, 2 processadores) sobre host windows (processador 11th i7-11800H, 2.3GHz, 16Gb ram). Foi utilizado o hipervisor [VMware Workstation PRO](https://www.vmware.com/br/products/workstation-pro/workstation-pro-evaluation.html) 17.0.0 build-20800274, e todas as vm's foram conectadas por rede personalizada VMnet8(NAT).

Certifique-se de ter o Docker (versão 24.0.2) instalado no seu sistema antes de executar esses comandos. Caso contrário, instale o docker engine de acordo com as instruções da [página do desenvolvedor](https://docs.docker.com/engine/install/ubuntu/).

## Instalação

1. Baixar a imagem Docker do OpenVPN:

   ```shell
      docker pull kylemanna/openvpn
   ```

2. Criar um volume Docker para persistir os dados do OpenVPN:

   ```shell
      docker volume create --name ovpn-data
   ```

## Configuração do servidor

### Gerar arquivos de configuração do servidor

3. Gerar o arquivo de configuração:

   ```shell
      docker run -v ovpn-data:/etc/openvpn --rm kylemanna/openvpn ovpn_genconfig -u udp://<ip-vm-host-do-docker>
   ```

4. Inicializar a infraestrutura de chave pública:

   ```shell
      docker run -v ovpn-data:/etc/openvpn --rm -it kylemanna/openvpn ovpn_initpki
   ```

Será iniciado o processo de configuração e geração de chaves e certificados para um servidor OpenVPN usando o OpenSSL. Anote as senhas para não ter problemas nesta etapa. Será solicitado um "Common Name" para identificar o servidor ou entidade para o certificado. Insira o nome do servidor vpn que desejar.

### Gerar arquivos de configuração do cliente

5. Gerar as chaves para o cliente "cliente01":

   ```shell
      docker run -v ovpn-data:/etc/openvpn --rm -it kylemanna/openvpn easyrsa build-client-full cliente01 nopass
   ```

6. Salvar o arquivo de configuração do cliente "cliente01" localmente:

   ```shell
      docker run -v ovpn-data:/etc/openvpn --rm kylemanna/openvpn ovpn_getclient cliente01 > cliente01.ovpn
   ```

7. Gerar as chaves para o cliente "cliente02":

   ```shell
      docker run -v ovpn-data:/etc/openvpn --rm -it kylemanna/openvpn easyrsa build-client-full cliente02 nopass
   ```

8. Salvar o arquivo de configuração do cliente "cliente02" localmente:

   ```shell
      docker run -v ovpn-data:/etc/openvpn --rm kylemanna/openvpn ovpn_getclient cliente02 > cliente02.ovpn
   ```

### Inicialização do servidor

9. Para inicializar o servidor OpenVPN, execute o comando:

   ```shell
      docker run -v ovpn-data:/etc/openvpn --rm -p 1194:1194/udp --cap-add=NET_ADMIN kylemanna/openvpn
   ```

## Transferência dos arquivos de configuração

Abra um novo terminal e execute os seguintes comandos para transferir os arquivos de configuração do cliente para os respectivos hosts dos clientes:

9. Utilizando SCP na vm hospedeira do container:

   ```shell
      scp /~/cliente01.ovpn <user>@<ip-da-vm02>:/home/cliente01/
   ```

   ```shell
      scp /~/cliente02.ovpn <user>@<ip-da-vm03>:/home/cliente02/
   ```

## Configuração nos hosts

10. Na vm02 "cliente01":

    ```shell
       sudo openvpn --config /~/cliente01.ovpn
    ```

11. Na vm03 "cliente02":

    ```shell
       sudo openvpn --config /~/cliente02.ovpn
    ```

## Verificação dos túneis

12. Para verificar os túneis após a configuração, execute o seguinte comando:

    ```shell
       ifconfing
    ```

13. Se necessário, instale o net-tools:

    ```shell
       sudo apt update && sudo apt install -y net-tools
    ```

## REFERÊNCIAS


https://blog.ricardopereira.eu/2015/01/28/PT-vpn-configuration-with-docker/

https://medium.com/@gurayy/set-up-a-vpn-server-with-docker-in-5-minutes-a66184882c45

https://hub.docker.com/r/kylemanna/openvpn/

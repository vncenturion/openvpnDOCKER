# openvpnDOCKER

Configuração de vpn-server em docker

## Pré-requisitos

Certifique-se de ter o Docker instalado no seu sistema antes de executar esses comandos.

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
docker run -v ovpn-data:/etc/openvpn --rm kylemanna/openvpn ovpn_genconfig -u udp:192.168.13.4
```

4. Inicializar a infraestrutura de chave pública:

```shell
docker run -v ovpn-data:/etc/openvpn --rm -it kylemanna/openvpn ovpn_initpki
```

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

## Transferência dos arquivos de configuração

Execute os seguintes comandos para transferir os arquivos de configuração do cliente para os respectivos hosts:


9. Baixar a imagem Docker do OpenVPN:

```shell
scp /~/cliente01.ovpn cliente01@192.168.13.7:/home/cliente01/
```

10. Baixar a imagem Docker do OpenVPN:

```shell
scp /~/cliente02.ovpn cliente02@192.168.13.8:/home/cliente02/
```

## Configuração nos hosts

11. No host "cliente01":

```shell
sudo openvpn --config /home/cliente01/cliente01.ovpn
```

11. No host "cliente02":

```shell
sudo openvpn --config /home/cliente02/cliente02.ovpn
```

## Verificação dos túneis

Para verificar os túneis após a configuração, execute o seguinte comando:

```shell
ifconfing
```

Se necessário, instale o net-tools:

```shell
sudo apt update && sudo apt install -y net-tools
```

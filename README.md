# Pterodactyl Panel & Wings (Dockerized) para Terraria tModLoader

Este repositório contém a configuração necessária para provisionar um ambiente completo do **Pterodactyl** (Painel Web + Wings) utilizando exclusivamente contêineres Docker.

A arquitetura foi desenhada para manter todos os dados persistentes (banco de dados, configurações e arquivos dos servidores de jogos) isolados dentro da própria pasta do repositório, facilitando backups e migrações.

---

## 📂 Estrutura de Diretórios

Após a execução inicial, seu repositório terá a seguinte estrutura:

```text
meu-repositorio/
├── docker-compose.yml      # Configuração do Painel, MariaDB e Redis
├── data/                   # Criada automaticamente (Banco de dados e arquivos do Painel)
├── config_wings/           # Arquivos de comunicação e certificados do daemon Wings
└── servidores/             # Volumes reais dos servidores de jogos (tModLoader/Terraria)
```

---

# 🚀 Como Iniciar (Passo a Passo)

## 1. Inicializar o Painel Web

Certifique-se de ter o Docker e o Docker Compose instalados na sua VPS.

Navegue até a pasta do repositório e suba os contêineres do painel em segundo plano:

```bash
docker compose up -d
```

---

## 2. Criar o Usuário Administrador

Com os contêineres rodando, execute o comando abaixo para gerar o seu usuário de acesso ao painel web.

Siga as instruções que aparecerão no terminal.

```bash
docker compose exec panel php artisan p:user:make
```

---

## 3. Configurar o Node via Interface Web

Acesse o painel pelo navegador usando o IP da sua VPS ou domínio configurado.

Faça login e vá para as configurações administrativas (**ícone de engrenagem**).

### Crie uma Location

Exemplo:

```
VPS Local
```

### Crie um Node

Configure:

* Associe à Location criada.
* Em **FQDN**, insira o IP da máquina ou domínio.
* Defina os limites de RAM e Disco conforme sua VPS.
* **Importante:** mantenha o diretório do daemon como:

```text
/var/lib/pterodactyl
```

Depois:

1. Abra a aba **Allocations**.
2. Adicione o IP da máquina.
3. Adicione as portas que os jogos utilizarão.

Exemplo:

```
7777
```

---

## 4. Inicializar o Wings (Daemon)

Na interface web:

1. Abra a aba **Configuration** do Node.
2. Clique em **Generate Auto-Deploy Token**.

Antes de utilizar o token, crie as pastas necessárias e inicie o contêiner do Wings.

```bash
mkdir -p config_wings servidores

docker run -d \
  --name=pterodactyl-wings \
  --restart=always \
  -p 8080:8080 \
  -p 2022:2022 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /var/lib/docker/containers:/var/lib/docker/containers \
  -v "$(pwd)/config_wings:/etc/pterodactyl" \
  -v "$(pwd)/servidores:/var/lib/pterodactyl" \
  --privileged \
  ghcr.io/pterodactyl/wings:latest
```

Após o contêiner iniciar, execute o comando de configuração utilizando o token gerado pelo painel:

```bash
docker exec -it pterodactyl-wings wings configure --panel-url https://... [COLE O RESTANTE DO TOKEN AQUI]
```

Em seguida, reinicie o Wings:

```bash
docker restart pterodactyl-wings
```

---

# 🎮 Subindo o tModLoader

1. Baixe o JSON do Egg do tModLoader no repositório oficial da comunidade (`parkervcp/eggs`).

2. No painel administrativo:

   * Vá em **Nests**.
   * Crie um novo Nest chamado **Terraria**.
   * Clique em **Import Egg**.
   * Faça upload do arquivo JSON.

3. Depois:

   * Vá em **Servers → Create New**.
   * Selecione o Egg **tModLoader**.
   * Utilize a porta **7777**.

O Wings fará automaticamente o download da imagem Docker necessária e iniciará o servidor.

---

# ⚠️ Cuidados Importantes e Boas Práticas

## 🔐 Credenciais no `docker-compose.yml`

Antes de publicar este projeto no GitHub ou GitLab:

* **Nunca** envie senhas reais.
* Altere:

  * `MYSQL_ROOT_PASSWORD`
  * `MYSQL_PASSWORD`
* Ou utilize um arquivo `.env`.
* Adicione o `.env` ao `.gitignore`.

---

## 📁 Caminhos Absolutos no Wings

O Wings precisa acessar diretamente o daemon Docker da máquina hospedeira.

Por isso, utilize obrigatoriamente:

```bash
$(pwd)
```

Exemplo:

```bash
-v "$(pwd)/servidores:/var/lib/pterodactyl"
```

**Não utilize:**

```bash
./servidores
```

Caso contrário, o Docker não conseguirá criar corretamente os volumes dos servidores.

---

## 🔥 Firewall (UFW/Iptables)

Certifique-se de liberar as seguintes portas:

| Porta | Finalidade                  |
| ----: | --------------------------- |
|    80 | Painel Web (HTTP)           |
|   443 | Painel Web (HTTPS)          |
|  8080 | Comunicação do Wings        |
|  2022 | Acesso SFTP                 |
|  7777 | Servidor Terraria (TCP/UDP) |

---

## 📂 Permissões de Diretório

Caso o Wings encontre problemas para gravar arquivos na pasta `servidores`, pode ser necessário ajustar as permissões do diretório no host.

Na maioria dos casos, a opção:

```bash
--privileged
```

já resolve problemas relacionados ao namespace de usuários.

---

## ✅ Pronto!

Após concluir essas etapas, seu ambiente estará preparado para criar e gerenciar servidores **Terraria tModLoader** através do painel **Pterodactyl**, mantendo todos os dados persistentes dentro do próprio repositório para facilitar backups e migrações.

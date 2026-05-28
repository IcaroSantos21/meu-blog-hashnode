---
title: "Meu primeiro laboratório Linux: Subindo o Arch com Docker"
datePublished: 2026-05-28T19:10:34.724Z
cuid: cmppvbnj600221sqk89vzfz3n
slug: meu-primeiro-laborat-rio-linux-subindo-o-arch-com-docker
cover: https://cdn.hashnode.com/uploads/covers/6a188a90782587548325f645/bf39462e-7e3c-481a-a90c-f770901ccd49.png

---

Bom, decidi aprender de vez como o Linux funciona, de fato, desde o básico até quebrar o sistema com um script. Para isso, decidi usar as seguintes ferramentas:

## O Ambiente de Testes

### Docker

A grosso modo, vou usar o Docker para criar um computador novo. Isso garante que caso eu dê um comando que eu não devia, minha máquina principal se mantenha intacta.

### Arch Linux

O Arch vai ser a distro que eu vou usar para aprender. A distribuição não importa tanto, mas decidi escolher essa porque foi a distro com que mexi primeiro.

## Configuração do Ambiente

Primeiro, baixamos a imagem do Arch com o Docker:

```shell
docker pull archlinux
```

Depois, rodamos o container criando um volume local:

```shell
docker run -it \
 --name archlab \ 
 -v ~/DockerLabs/archlab:/workspace
 \ archlinux bash
```

**O que cada parte faz:**

*   `-it`: Ativa o modo interativo e o TTY (essencial para conseguir usar o terminal/bash).
    
*   `--name archlab`: Dá o nome de “archlab” para o container (facilita muito para dar o `docker start` depois).
    
*   `-v ~/DockerLabs/archlab:/workspace`: Vincula a pasta da minha máquina real (`~/` representa a sua pasta de usuário Home) com a pasta dentro do container (`/workspace`). Tudo que eu alterar em uma muda na outra.
    
*   `archlinux`: A imagem oficial que estou usando.
    

`bash`: O comando que será executado assim que o container ligar.Primeiros Passos no Terminal

Vamos começar pela parte inicial de todo Linux: atualizar todos os pacotes do sistema.

Bash

```plaintext
pacman -Syu # Sincronizando e atualizando os pacotes
```

Esse comando é o equivalente ao `apt update && apt upgrade` das distribuições baseadas em Ubuntu/Debian. Vamos destrinchar o que cada letra faz:

*   **Pacman** → É o gerenciador de pacotes oficial do Arch Linux. Ele é o responsável por instalar, remover e atualizar os softwares do sistema.
    
*   **\-S** → Vem de *Sync*. Ele diz para o pacman sincronizar ou operar com a base de dados oficial de pacotes da internet. Sempre que precisar instalar ou atualizar algo grandioso, o `-S` (maiúsculo) entra em ação.
    
*   **y** → Faz o pacman baixar a lista mais recente de programas que existem nos servidores do Arch Linux.
    
*   **u** → Compara a lista que acabou de baixar com os programas que já estão instalados na máquina. Se ele notar que algo está desatualizado, ele baixa e instala as versões mais novas.
    

Basicamente, quando eu uso esse comando, eu estou dizendo:

> *“Ei pacman, conecta nos servidores (-S), baixe a lista de atualizações mais recentes (y) e atualiza tudo o que estiver velho no meu sistema (u).”*

## Instalando Ferramentas Essenciais

Agora vou instalar as ferramentas que vou precisar neste começo de aprendizado:

Bash

```plaintext
pacman -S vim git curl wget man-db man-pages base-devel
```

Vamos passar rapidamente pelo papel de cada uma:

*   **Vim:** Editor de texto direto no terminal. No Linux usamos muito para editar configurações, scripts, código e serviços.
    
*   **Git:** Controle de versão. Essencial para criar o histórico do código, trabalhar com branches e garantir segurança para o projeto.
    
*   **curl:** Ferramenta para fazer requisições HTTP. Extremamente importante para quem estuda backend, automação e quer mexer com APIs (Ex: `curl` [`https://api.github.com`](https://api.github.com)).
    
*   **wget:** Um baixador de arquivos direto pelo terminal, ideal para puxar pacotes `.zip` ou `.tar.gz`.
    
*   **man-db & man-pages:** Ativam o sistema de manuais e documentação do Linux. Se eu tiver dúvida de como usar um comando, basta rodar `man pacman` para abrir a documentação oficial na tela.
    
*   **base-devel:** Um grupo de ferramentas essenciais para compilação. Ele inclui ferramentas como `gcc`, `make` e `fakeroot`. É o que resolve o problema de "como compilar programas" e será muito útil no futuro para usar o AUR (Arch User Repository).
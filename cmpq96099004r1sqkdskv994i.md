---
title: "Lab Linux #02: Entendendo a Raiz do Sistema e o Poder das Permissões"
datePublished: 2026-05-29T01:38:05.904Z
cuid: cmpq96099004r1sqkdskv994i
slug: lab-linux-02-entendendo-a-raiz-do-sistema-e-o-poder-das-permiss-es
cover: https://cdn.hashnode.com/uploads/covers/6a188a90782587548325f645/12db0162-890d-4463-b9f4-fd36ef65d1bc.png

---

## Core do Sistema: O File System (Sistema de Arquivos)

Como passei a vida inteira usando Windows, estava totalmente acostumado com a estrutura clássica: `C:\`, `Program Files`, `System32` e `Users`. Quando mudei de distribuição no Linux pela primeira vez, confesso que fiquei um pouco perdido.

Para resolver isso de vez, vamos navegar pelas pastas do nosso container para entender o que cada uma representa. No Linux, existe uma máxima clássica: **"Tudo é um arquivo"** (até mesmo os componentes de hardware). Em vez de discos separados (`C:`, `D:`), o sistema de arquivos do Linux é uma árvore única que nasce a partir da raiz, representada por uma barra: `/`.

Pesquisando um pouco, descobri as pastas que preciso dominar imediatamente:

*   `/etc` : Onde ficam os arquivos de configuração do sistema (configurações de rede, usuários, serviços, etc.).
    
*   `/var/log` : Onde o sistema e os aplicativos guardam os logs (o histórico de tudo o que acontece na máquina).
    
*   `/home` : A pasta dos usuários comuns. No nosso ambiente de testes, nós criamos a pasta `/workspace` mapeada direto com a máquina real, o que substitui essa necessidade no momento e protege nossos códigos.
    

### Investigando os Usuários do Sistema

Para ver isso na prática, podemos abrir um arquivo de configuração vital usando o editor que instalamos:

```shell
vim /etc/passwd
```

Esse comando abre o arquivo que lista os usuários configurados no sistema. Ele retorna uma lista longa, parecida com esta:

```shell
root:x:0:0::/root:/usr/bin/bash
alpm:x:978:978:Arch Linux Package Management:/:/usr/bin/nologin
bin:x:1:1::/:/usr/bin/nologin
daemon:x:2:2::/:/usr/bin/nologin
mail:x:8:12::/var/spool/mail:/usr/bin/nologin
git:x:969:969:git daemon user:/:/usr/bin/git-shell
```

Parece confuso à primeira vista, mas cada linha representa um usuário e é dividida em **7 campos** separados por dois-pontos (`:`). Vamos destrinchar a linha do `root`:

`root:x:0:0::/root:/usr/bin/bash`

1.  **root** (Nome de usuário): O nome usado para autenticação.
    
2.  **x** (Senha): Antigamente a senha ficava visível aqui. Hoje, por segurança, elas ficam criptografadas no arquivo `/etc/shadow`. O `x` apenas indica que há uma senha salva.
    
3.  **0** (UID - User ID): A identidade numérica do usuário para o sistema. O `root` sempre será o `0`.
    
4.  **0** (GID - Group ID): O ID do grupo principal ao qual esse usuário pertence.
    
5.  **Campos vazios** `::` (Gecos): Espaço para o nome real ou notas. No root está vazio, mas repare que no usuário `alpm` está escrito *"Arch Linux Package Management"*.
    
6.  **/root** (Diretório Home): A pasta pessoal deste usuário.
    
7.  **/usr/bin/bash** (Shell de Inicialização): O terminal padrão que se abre quando o usuário faz login.
    

**O que realmente importa aprender aqui:**

*   **O Superusuário (root):** É o chefe supremo. Ele tem o UID `0`, usa o bash e pode criar ou apagar absolutamente qualquer coisa.
    
*   **Usuários de Sistema (**`daemon`**,** `http`**,** `git`**):** Eles existem por segurança. Se um hacker invadir um serviço web que roda sob o usuário `http`, ele só terá acesso aos arquivos desse serviço, ficando isolado do resto do sistema.
    
*   **A trava** `/usr/bin/nologin`**:** A maioria dos usuários do sistema termina com isso. Significa que ninguém pode fazer login no terminal usando essas contas; elas servem apenas para rodar processos em segundo plano.
    

## Linha de Defesa: Permissões de Arquivos

Como o Linux nasceu para ser um sistema multiusuário (onde várias pessoas usam o mesmo servidor ao mesmo tempo), as permissões controlam rigidamente quem pode fazer o quê. Isso impede que um usuário quebre o sistema ou bisbilhote os arquivos do outro.

Cada arquivo e diretório pertence a um **Usuário** e a um **Grupo**, e suas permissões são divididas em três ações básicas:

| Sigla | Significado | O que faz em Arquivos | O que faz em Pastas (Diretórios) |
| --- | --- | --- | --- |
| **r** | Read (Leitura) | Permite ler o conteúdo do arquivo. | Permite listar os arquivos de dentro dela (`ls`). |
| **w** | Write (Escrita) | Permite modificar o arquivo. | Permite criar ou deletar arquivos dentro dela. |
| **x** | Execute (Execução) | Permite rodar o arquivo (se for um script). | Permite entrar na pasta (`cd`). |

### Criando um Laboratório de Testes (Script Prático)

Para entender como isso funciona na pele, criei um script simples dentro do container que testa essas restrições. Você pode criá-lo rodando o comando abaixo no terminal:

```shell
cat << 'EOF' > testar_permissoes.sh
#!/bin/bash

ARQUIVO="dados_secretos.txt"

echo "=== 1. CRIANDO O ARQUIVO ==="
echo "Texto original muito importante" > "$ARQUIVO"
echo "Arquivo criado."
ls -l "$ARQUIVO"

echo -e "\n=== 2. REMOVENDO PERMISSÃO DE LEITURA (r) ==="
chmod -r "$ARQUIVO"
ls -l "$ARQUIVO"

echo -e "\n=== Tentando ler o arquivo sem permissão 'r': ==="
cat "$ARQUIVO"

echo -e "\n=== 3. DEVOLVENDO LEITURA E REMOVENDO ESCRITA (w) ==="
chmod +r-w "$ARQUIVO"
ls -l "$ARQUIVO"

echo -e "\n=== Tentando alterar o arquivo sem permissão 'w': ==="
echo "Tentando quebrar o sistema" >> "$ARQUIVO"

echo -e "\n=== 4. RESTAURANDO TUDO ==="
chmod +rw "$ARQUIVO"
echo "Conteúdo final do arquivo:"
cat "$ARQUIVO"
EOF
```

Se tentarmos executar esse arquivo imediatamente com `./testar_`[`permissoes.sh`](http://permissoes.sh), o Linux nos dá uma resposta bem objetiva: `bash: ./testar_`[`permissoes.sh`](http://permissoes.sh)`: Permission denied`

### O motivo do bloqueio

Por padrão, quando criamos um arquivo de texto, ele não vem com permissão de execução (`x`). Para verificar isso, usamos o comando:

```shell
ls -l testar_permissoes.sh
```

O retorno será algo como: `-rw-r--r-- 1 root root 681 May 29 01:09 testar_`[`permissoes.sh`](http://permissoes.sh) (indicando apenas leitura e escrita).

### Dando superpoderes ao Script

Para mudar isso, usamos o comando `chmod` (Change Mode) para adicionar (`+`) a permissão de execução (`x`):

```shell
chmod +x testar_permissoes.sh
```

Se listarmos o arquivo novamente, ele terá mudado de cor no terminal e mostrará os novos atributos: `-rwxr-xr-x 1 root root 681 May 29 01:09 testar_`[`permissoes.sh`](http://permissoes.sh)

Agora, ao rodar `./testar_`[`permissoes.sh`](http://permissoes.sh), o script executa o nosso laboratório!

### ⚠️ Uma observação crucial sobre o Docker

Se você estiver rodando esse container como usuário `root` (o padrão do Docker, onde o prompt do terminal mostra um `#` em vez de `$`), você vai notar algo estranho: **o script vai rodar inteiro sem dar nenhum erro de "Permission denied" nos testes internos.**

Por quê? Porque o `root` é o "Deus" do sistema. Ele é tão poderoso que ignora as restrições de leitura (`r`) e escrita (`w`) dos arquivos comuns. O único bloqueio que o `root` respeita é o de execução (`x`) — que foi exatamente o erro que recebemos antes do `chmod +x`.

Para que esse teste de permissões funcione perfeitamente e exiba os erros controlados de leitura e escrita na tela, o próximo passo ideal do nosso laboratório será aprender a **criar um usuário comum** dentro do sistema e alternar para ele.
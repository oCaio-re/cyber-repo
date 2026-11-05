# cyber-repo

Material de estudo de cibersegurança para iniciantes: um roadmap de 12 meses com conteúdo, exercícios e projetos, e um guia para montar o laboratório no **Linux Mint**.

| Arquivo | O que é |
|---|---|
| `roadmap-ciberseguranca.html` | Roadmap interativo. Abra no navegador e marque os exercícios conforme concluir. |
| `README.md` | Este guia de instalação. |

> **Aviso legal:** use as ferramentas daqui apenas no seu próprio laboratório, em plataformas de CTF ou em programas de bug bounty, sempre dentro do escopo autorizado. Invadir dispositivo alheio é crime no Brasil (Lei 12.737/2012).

---

## 1. Requisitos

- Linux Mint 21 ou 22 (as versões baseadas em Ubuntu).
- Pelo menos **8 GB de RAM** (16 GB é o ideal para rodar duas VMs ao mesmo tempo).
- Pelo menos **60 GB livres** em disco.
- Virtualização ativada na BIOS/UEFI (a opção se chama **Intel VT-x** ou **AMD-V/SVM**).

Para conferir se a virtualização está ativa, abra o Terminal (`Ctrl+Alt+T`) e rode:

```bash
egrep -c '(vmx|svm)' /proc/cpuinfo
```

Se o resultado for `0`, entre na BIOS (geralmente pressionando `F2`, `F10` ou `Del` ao ligar o PC) e ative a virtualização.

---

## 2. Atualizar o sistema e instalar o básico

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y git curl wget vim build-essential p7zip-full \
  python3 python3-pip python3-venv
```

Configure seu nome e e-mail no Git (eles aparecem nos seus commits):

```bash
git config --global user.name "Seu Nome"
git config --global user.email "seu-email@exemplo.com"
```

---

## 3. Clonar o repositório

```bash
mkdir -p ~/estudos && cd ~/estudos
git clone https://github.com/oCaio-re/cyber-repo.git
cd cyber-repo
```

Para baixar as atualizações depois, rode `git pull` dentro da pasta.

Para abrir o roadmap no navegador:

```bash
xdg-open roadmap-ciberseguranca.html
```

O progresso que você marca nas caixas fica salvo só nesse navegador.

---

## 4. Ferramentas no próprio Mint

Estas ferramentas são usadas nas fases 1 e 2 do roadmap e rodam direto no seu sistema:

```bash
sudo apt install -y nmap wireshark tcpdump netcat-openbsd \
  john hashcat whois dnsutils openssl gnupg
```

Durante a instalação do Wireshark aparece a pergunta *"Should non-superusers be able to capture packets?"*. Responda **Sim**. Depois, adicione seu usuário ao grupo:

```bash
sudo usermod -aG wireshark $USER
```

### Ambiente Python para os projetos

Crie um ambiente virtual para não misturar as bibliotecas dos projetos com as do sistema:

```bash
cd ~/estudos/cyber-repo
python3 -m venv .venv
source .venv/bin/activate
pip install requests
```

Rode `source .venv/bin/activate` sempre que abrir um terminal novo para trabalhar nos projetos. Para sair do ambiente, digite `deactivate`.

---

## 5. VirtualBox

```bash
sudo apt install -y virtualbox virtualbox-ext-pack
sudo usermod -aG vboxusers $USER
```

**Faça logout e login de novo** (ou reinicie) para que os grupos `vboxusers` e `wireshark` passem a valer.

> **Secure Boot:** se o VirtualBox reclamar que o módulo do kernel não carregou, é provável que o Secure Boot esteja ativo. Ao instalar, o Mint pede uma senha para registrar a chave (MOK); na próxima inicialização escolha **Enroll MOK** e digite essa senha. A alternativa é desativar o Secure Boot na BIOS.

### Criar a rede isolada do laboratório

As máquinas vulneráveis **nunca** devem ficar acessíveis pela internet ou pela sua rede de casa. Use uma rede *host-only*:

1. Abra o VirtualBox e vá em **Arquivo → Ferramentas → Gerenciador de Rede**.
2. Na aba **Redes Exclusivas de Hospedeiro (Host-only)**, clique em **Criar**.
3. Deixe o endereço padrão (`192.168.56.1/24`) com o servidor DHCP ativado.

---

## 6. Máquinas virtuais

### Kali Linux (máquina atacante)

1. Baixe a imagem pronta para **VirtualBox** em <https://www.kali.org/get-kali/#kali-virtual-machines>.
2. Extraia o arquivo:
   ```bash
   cd ~/Downloads
   7z x kali-linux-*-virtualbox-amd64.7z
   ```
3. No VirtualBox, clique em **Adicionar** e selecione o arquivo `.vbox` extraído.
4. Em **Configurações → Rede**, deixe o **Adaptador 1** como **NAT** (para ter internet) e ative o **Adaptador 2** como **Placa de rede exclusiva de hospedeiro** (para falar com o laboratório).
5. Usuário e senha padrão: `kali` / `kali`. Troque a senha no primeiro login com `passwd`.

O Kali já vem com Burp Suite, Metasploit, gobuster, ffuf e o resto das ferramentas das fases 3 e 5.

### Metasploitable 2 (máquina alvo)

1. Baixe em <https://sourceforge.net/projects/metasploitable/>.
2. Extraia o `.zip`, crie uma VM nova (tipo **Linux**, versão **Ubuntu 32-bit**, 512 MB de RAM) e, em vez de criar um disco, use o arquivo `.vmdk` que veio no zip.
3. Em **Rede**, use **somente** a placa **exclusiva de hospedeiro**. Nada de NAT nem bridge: essa máquina é vulnerável de propósito.
4. Login: `msfadmin` / `msfadmin`.

### Snapshots

Antes de cada exercício, tire um snapshot (**Máquina → Criar Snapshot**). Se algo quebrar, é só restaurar.

---

## 7. Aplicações web vulneráveis com Docker (fase 3)

```bash
sudo apt install -y docker.io
sudo usermod -aG docker $USER
```

Faça logout e login de novo para usar o Docker sem `sudo`. Depois, suba as aplicações:

```bash
# OWASP Juice Shop -> http://localhost:3000
docker run -d --name juice-shop -p 127.0.0.1:3000:3000 bkimminich/juice-shop

# DVWA -> http://localhost:8080  (login: admin / password)
docker run -d --name dvwa -p 127.0.0.1:8080:80 vulnerables/web-dvwa
```

O `127.0.0.1:` no início da porta faz as aplicações só aceitarem conexões do seu próprio computador, e não da rede. Não remova isso.

Para parar e iniciar depois:

```bash
docker stop juice-shop dvwa
docker start juice-shop dvwa
```

No DVWA, clique em **Create / Reset Database** na primeira vez que abrir.

---

## 8. Checklist final

Se todos os comandos abaixo funcionarem, o ambiente está pronto:

```bash
git --version
python3 --version
nmap --version
groups | grep -E 'wireshark|vboxusers|docker'   # os três grupos devem aparecer
docker ps                                       # deve listar juice-shop e dvwa
```

E também:

- [ ] O roadmap abre no navegador.
- [ ] A VM do Kali inicia e acessa a internet.
- [ ] No Kali, `ping 192.168.56.X` responde (troque `X` pelo IP do Metasploitable, que aparece ao rodar `ip a` dentro dele).
- [ ] <http://localhost:3000> e <http://localhost:8080> abrem no navegador do Mint.

---

## 9. Problemas comuns

| Problema | Solução |
|---|---|
| `permission denied` ao rodar `docker` | Você não fez logout/login depois do `usermod`. Reinicie a sessão. |
| Wireshark não mostra nenhuma interface | Mesmo motivo: o grupo `wireshark` ainda não vale na sua sessão. |
| VM não inicia, erro de `VT-x` ou `AMD-V` | Ative a virtualização na BIOS (seção 1). |
| VirtualBox: `Kernel driver not installed` | Secure Boot ativo. Veja a nota na seção 5. |
| Kali e Metasploitable não se enxergam | Confira se as duas VMs estão na mesma rede host-only (`vboxnet0`). |
| PC lento com as VMs abertas | Dê no máximo metade da RAM às VMs e rode uma de cada vez. |

---

## Próximos passos

Abra o `roadmap-ciberseguranca.html` e comece pela **Fase 1**. Uma sugestão para organizar seu trabalho neste repositório:

```
cyber-repo/
├── notas/       # anotações e write-ups em Markdown
└── projetos/    # um subdiretório para cada projeto do roadmap
```

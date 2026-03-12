# 🐧 Fedora Silverblue — Guia de Pós-Instalação Manual

**Data:** 12/03/2026  
**Ambiente:** Fedora Silverblue (Fedora 43)  
**Hostname:** (será configurado por você)

---

## ⚠️ IMPORTANTE: Ordem de Execução e Reboots

Este guia está **organizado por ordem de execução**. Siga **exatamente** a ordem descrita. Há **3 pontos de reboot obrigatórios**:

1. ✅ **Reboot 1** - Após instalar RPM Fusion
2. ✅ **Reboot 2** - Após instalar Codecs/VA-API
3. ✅ **Reboot 3** - Após instalar TLP (opcional, recomendado)

---

# FASE 1: Remoções (Sem reboot)

## 1️⃣ Remover Flatpaks Pré-Instalados

Execute um de cada vez:

```bash
flatpak uninstall --noninteractive org.fedoraproject.MediaWriter
flatpak uninstall --noninteractive org.gnome.Calendar
flatpak uninstall --noninteractive org.gnome.Characters
flatpak uninstall --noninteractive org.gnome.Connections
flatpak uninstall --noninteractive org.gnome.Contacts
flatpak uninstall --noninteractive org.gnome.Extensions
flatpak uninstall --noninteractive org.gnome.Maps
flatpak uninstall --noninteractive org.gnome.Snapshot
flatpak uninstall --noninteractive org.gnome.Weather
flatpak uninstall --noninteractive org.gnome.clocks
```

Limpar runtimes não utilizados:

```bash
flatpak uninstall --unused --noninteractive
```

---

## 2️⃣ Remover Apps do Sistema Base

⚠️ **Cada remoção via `rpm-ostree` cria um novo deployment. Isso é normal.**

### Remover GNOME Tour:

```bash
# Verificar se está instalado:
rpm -q gnome-tour

# Se sim, remover:
rpm-ostree override remove gnome-tour
```

Espere a operação terminar (pode levar alguns segundos).

### Remover Help (Yelp):

```bash
# Verificar:
rpm -q yelp

# Remover:
rpm-ostree override remove yelp
```

### Remover Parental Controls (UI):

```bash
# Verificar:
rpm -q malcontent-control

# Remover:
rpm-ostree override remove malcontent-control malcontent-ui-libs
```

✅ **Fim da Fase 1 (sem reboot necessário)**

---

# FASE 2: Configurações do Sistema e RPM Fusion

## 3️⃣ Adicionar Flathub

```bash
flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
```

Responda `y` se perguntado.

---

## 4️⃣ Instalar RPM Fusion

### Adicionar RPM Fusion Free:

```bash
sudo rpm-ostree install --idempotent \
  https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm
```

Aguarde a conclusão (1-2 minutos).

### Adicionar RPM Fusion Non-Free:

```bash
sudo rpm-ostree install --idempotent \
  https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm
```

Aguarde a conclusão.

---

## 5️⃣ Definir Hostname

### Passo 1: Definir Variável (para uso nesta sessão)

```bash
# Define o hostname desejado em uma variável
# SUBSTITUA 'seu-hostname' pelo seu hostname desejado
MYHOSTNAME="seu-hostname"

# Verificar a variável:
echo "Hostname será: $MYHOSTNAME"
```

### Passo 2: Aplicar Hostname ao Sistema

```bash
# Usar a variável para definir o hostname:
hostnamectl set-hostname "$MYHOSTNAME"
```

### Passo 3: Verificar Configuração

```bash
# Verificar que foi aplicado:
hostnamectl

# Verificar a variável ainda existe:
echo "Hostname definido como: $MYHOSTNAME"
```

**Nota:** Se você fechar o terminal antes de Reboot 1, a variável se perdará. Não é problema pois o hostname já foi aplicado ao sistema via `hostnamectl`.

---

## 6️⃣ Atualizar Sistema

```bash
rpm-ostree upgrade
```

---

## 🔴 REBOOT 1 - OBRIGATÓRIO

RPM Fusion foi instalado e precisa ser ativado no novo deployment.

```bash
systemctl reboot
```

**Espere o sistema reiniciar e volte ao login.**

✅ **Após reboot: RPM Fusion está ATIVO**

---

# FASE 3: Codecs e Aceleração de Vídeo

## 7️⃣ Instalar/Atualizar Codecs FFmpeg

Execute **exatamente nesta ordem**:

### Passo 1: Detectar Status

```bash
echo "Status do FFmpeg:"
rpm -q ffmpeg-free
rpm -q ffmpeg
rpm -q gstreamer1-plugins-bad-freeworld
rpm -q gstreamer1-plugin-libav
```

### Passo 2: Executar Ação Apropriada

**Se tem `ffmpeg-free` e NÃO tem `ffmpeg`:**

```bash
sudo rpm-ostree override remove \
  libavdevice-free libavcodec-free libavfilter-free libavformat-free \
  libavutil-free libpostproc-free libswresample-free libswscale-free \
  ffmpeg-free \
  --install ffmpeg \
  --install --allow-inactive gstreamer1-plugin-libav \
  --install --allow-inactive gstreamer1-plugins-bad-free-extras \
  --install --allow-inactive gstreamer1-plugins-bad-freeworld \
  --install --allow-inactive gstreamer1-plugins-ugly \
  --install --allow-inactive mozilla-openh264 \
  --install --allow-inactive gstreamer1-plugin-openh264
```

**Se já tem `ffmpeg` mas faltam gstreamer:**

```bash
sudo rpm-ostree install --idempotent --allow-inactive \
  gstreamer1-plugin-libav \
  gstreamer1-plugins-bad-free-extras \
  gstreamer1-plugins-bad-freeworld \
  gstreamer1-plugins-ugly \
  mozilla-openh264 \
  gstreamer1-plugin-openh264
```

**Se não tem nenhum:**

```bash
sudo rpm-ostree install --idempotent --allow-inactive \
  ffmpeg \
  gstreamer1-plugin-libav \
  gstreamer1-plugins-bad-free-extras \
  gstreamer1-plugins-bad-freeworld \
  gstreamer1-plugins-ugly \
  mozilla-openh264 \
  gstreamer1-plugin-openh264
```

Aguarde a conclusão (2-3 minutos).

---

## 8️⃣ Instalar Intel VA-API (Aceleração de Vídeo)

### Passo 1: Detectar Hardware

```bash
# Detectar qual GPU você tem:
lspci | grep -E "Intel|AMD|NVIDIA"
```

### Passo 2: Verificar Status

```bash
echo "Status de VA-API:"
rpm -q intel-media-driver
rpm -q mesa-va-drivers-freeworld
rpm -q mesa-va-drivers
rpm -q libva-utils
```

### Passo 3: Instalar Apropriadamente

**Se tem apenas `mesa-va-drivers` (base/livre):**

```bash
sudo rpm-ostree override remove mesa-va-drivers \
  --install mesa-va-drivers-freeworld \
  --install intel-media-driver \
  --install libva-utils
```

**Se não tem nenhum:**

```bash
sudo rpm-ostree install --idempotent --allow-inactive \
  intel-media-driver \
  mesa-va-drivers-freeworld \
  libva-utils
```

**Se já tem tudo instalado:**

Passe para o próximo passo.

---

## 🔴 REBOOT 2 - OBRIGATÓRIO

Codecs e VA-API foram instalados no overlay.

```bash
systemctl reboot
```

**Espere o sistema reiniciar.**

✅ **Após reboot: Codecs e VA-API estão ATIVOS**

---

# FASE 4: Instalação de Pacotes

## 9️⃣ Instalar VS Code

### Passo 1: Adicionar Repositório Microsoft

```bash
sudo sh -c 'echo -e "[code]\nname=Visual Studio Code\nbaseurl=https://packages.microsoft.com/yumrepos/vscode\nenabled=1\ngpgcheck=1\ngpgkey=https://packages.microsoft.com/keys/microsoft.asc" > /etc/yum.repos.d/vscode.repo'
```

Aguarde a conclusão (apenas cria arquivo de configuração).

### Passo 2: Instalar VS Code

```bash
sudo rpm-ostree install --idempotent code
```

Aguarde a conclusão (2-3 minutos).

### Verificar Instalação

```bash
code --version
```

Deve exibir versão do VS Code.

---

## 🔟 Instalar TLP (Gerenciamento de Bateria)

```bash
sudo rpm-ostree install --idempotent --allow-inactive tlp tlp-rdw
```

---

## 1️⃣1️⃣ Instalar Pacotes RPM Overlay

```bash
sudo rpm-ostree install --idempotent --allow-inactive \
  zsh \
  fastfetch \
  git \
  curl \
  wget \
  unzip \
  p7zip \
  gnome-tweaks \
  fzf \
  bat
```

Aguarde a conclusão (2-3 minutos).

---

## 🔴 REBOOT 3 - RECOMENDADO

```bash
systemctl reboot
```

---

# FASE 5: Configurações GNOME

## 1️⃣2️⃣ Configurar Tema e Preferências GNOME

### Ativar Dark Mode:

```bash
gsettings set org.gnome.desktop.interface color-scheme 'prefer-dark'
```

### Mostrar Botões Minimizar/Maximizar:

```bash
gsettings set org.gnome.desktop.wm.preferences button-layout 'appmenu:minimize,maximize,close'
```

### Ativar Tap to Click:

```bash
gsettings set org.gnome.desktop.peripherals.touchpad tap-to-click true
```

### Centralizar Novas Janelas:

```bash
gsettings set org.gnome.mutter center-new-windows true
```

### Melhorar Font Hinting:

```bash
gsettings set org.gnome.desktop.interface font-hinting 'full'
```

### Desativar Hot Corner:

```bash
gsettings set org.gnome.desktop.interface enable-hot-corners false
```

---

# FASE 6: Configuração Git

## 1️⃣3️⃣ Configurar Git Globalmente

### Passo 1: Verificar Git Instalado

```bash
# Verificar que Git está instalado:
command -v git

# Se não estiver, instalar:
sudo rpm-ostree install --idempotent git
```

### Passo 2: Definir Variáveis para Git

```bash
# Define variáveis que serão usadas nesta sessão
# SUBSTITUA pelos seus dados:
GIT_NAME="Seu Nome Completo"
GIT_EMAIL="seu.email@exemplo.com"

# Verificar as variáveis:
echo "Nome Git: $GIT_NAME"
echo "Email Git: $GIT_EMAIL"
```

### Passo 3: Configurar Identidade com Variáveis

```bash
# Usar as variáveis para configurar:
git config --global user.name "$GIT_NAME"
git config --global user.email "$GIT_EMAIL"

# Verificar que foram aplicadas:
echo "Configurado Git para: $GIT_NAME <$GIT_EMAIL>"
git config --global user.name
git config --global user.email
```

### Passo 4: Configurar Comportamento

```bash
git config --global init.defaultBranch main
git config --global pull.rebase true
git config --global core.editor "code --wait"
```

### Passo 5: Configurar Aliases Úteis

```bash
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.lg "log --oneline --graph --decorate --all"
```

### Passo 6: Verificar Configuração Completa

```bash
# Ver todas as configurações:
git config --global --list

# Ou verificar específicas:
echo "Git Name: $(git config --global user.name)"
echo "Git Email: $(git config --global user.email)"
echo "Default Branch: $(git config --global init.defaultBranch)"
echo "Pull Rebase: $(git config --global pull.rebase)"
echo "Editor: $(git config --global core.editor)"
```

**Nota:** As variáveis `$GIT_NAME` e `$GIT_EMAIL` foram usadas para configurar o Git. Se fechar o terminal, as variáveis se perdem, mas as configurações do Git permanecem (foram salvas em `~/.gitconfig`).

---

# FASE 7: Instalar Flatpaks (Aplicativos)

## 1️⃣4️⃣ Instalar Aplicativos via Flatpak

Execute um de cada vez (cada instalação leva 1-3 minutos):

```bash
# Google Chrome
flatpak install --noninteractive flathub com.google.Chrome

# Spotify
flatpak install --noninteractive flathub com.spotify.Client

# Discord
flatpak install --noninteractive flathub com.discordapp.Discord

# Bitwarden (Gerenciador de Senhas)
flatpak install --noninteractive flathub com.bitwarden.desktop

# Boxes (Máquinas Virtuais)
flatpak install --noninteractive flathub org.gnome.Boxes

# Extension Manager
flatpak install --noninteractive flathub com.mattjakeman.ExtensionManager

# Flameshot (Screenshot)
flatpak install --noninteractive flathub org.flameshot.Flameshot

# Flatseal (Permissões Flatpak)
flatpak install --noninteractive flathub com.github.tchx84.Flatseal

# VLC (Reprodutor de Mídia)
flatpak install --noninteractive flathub org.videolan.VLC
```

---

# FASE 8: Ferramentas de Desenvolvimento

## 1️⃣5️⃣ Instalar eza (ls moderno)

### Criar diretório:

```bash
mkdir -p "$HOME/.local/bin"
```

### Obter versão mais recente:

```bash
EZA_RELEASE=$(curl -s https://api.github.com/repos/eza-community/eza/releases/latest 2>/dev/null | grep "tag_name" | cut -d'"' -f4)
echo "Versão do eza: $EZA_RELEASE"
```

### Montar URL:

```bash
EZA_URL="https://github.com/eza-community/eza/releases/download/${EZA_RELEASE}/eza_x86_64-unknown-linux-gnu.tar.gz"
echo "URL: $EZA_URL"
```

### Download e instalação:

```bash
wget -qO- "$EZA_URL" 2>/dev/null | tar xz -C "$HOME/.local/bin" eza 2>/dev/null
chmod +x "$HOME/.local/bin/eza"

# Verificar:
$HOME/.local/bin/eza --version
```

### Adicionar ao PATH

Edite `~/.bashrc` ou `~/.zshrc`:

```bash
# Adicionar esta linha ao final:
export PATH="$HOME/.local/bin:$PATH"
```

Depois, recarregar shell:

```bash
source ~/.bashrc  # ou ~/.zshrc se estiver usando zsh
```

Verificar:

```bash
eza --version
```

---

# FASE 9: Instalar Fontes (Typefaces)

## 1️⃣6️⃣ Criar Diretório de Fontes

```bash
mkdir -p "$HOME/.local/share/fonts"
```

---

## 1️⃣7️⃣ Instalar Inter Font

**Inter** - Fonte sans-serif moderna, excelente para UI/UX

```bash
# Criar diretório temporário:
mkdir -p /tmp/fonts && cd /tmp/fonts

# Download Inter (versão mais recente):
wget -q https://github.com/rsms/inter/releases/download/v4.0/Inter-4.0.zip

# Descompactar:
unzip -q Inter-4.0.zip

# Copiar apenas arquivos .otf para fonts:
cp Inter-4.0/InterTight/OTF/*.otf "$HOME/.local/share/fonts/"

# Verificar:
ls "$HOME/.local/share/fonts/" | grep -i inter
```

---

## 1️⃣8️⃣ Instalar JetBrains Mono (Nerd Font)

**JetBrains Mono Nerd Font** - Monospace com ícones Nerd Font integrados (ótima para terminal/editor)

```bash
# Criar diretório temporário:
cd /tmp/fonts

# Download JetBrains Mono Nerd Font:
wget -q https://github.com/ryanoasis/nerd-fonts/releases/download/v3.0.2/JetBrainsMono.zip

# Descompactar:
unzip -q JetBrainsMono.zip -d JetBrainsMono

# Copiar .ttf para fonts:
cp JetBrainsMono/*.ttf "$HOME/.local/share/fonts/"

# Verificar:
ls "$HOME/.local/share/fonts/" | grep -i jetbrains
```

---

## 1️⃣9️⃣ Instalar Cascadia Code

**Cascadia Code** - Fonte monospace moderna do Microsoft (ótima para código)

```bash
# Criar diretório temporário:
cd /tmp/fonts

# Download Cascadia Code:
wget -q https://github.com/microsoft/cascadia-code/releases/download/v2404.27/CascadiaCode-2404.27.zip

# Descompactar:
unzip -q CascadiaCode-2404.27.zip -d CascadiaCode

# Copiar .ttf/.otf para fonts:
cp CascadiaCode/ttf/*.ttf "$HOME/.local/share/fonts/"
cp CascadiaCode/otf/*.otf "$HOME/.local/share/fonts/"

# Verificar:
ls "$HOME/.local/share/fonts/" | grep -i cascadia
```

---

## 2️⃣0️⃣ Instalar Fira Code

**Fira Code** - Monospace com ligaduras (ideal para desenvolvedores)

```bash
# Criar diretório temporário:
cd /tmp/fonts

# Download Fira Code:
wget -q https://github.com/tonsky/FiraCode/releases/download/6.2/Fira_Code_v6.2.zip

# Descompactar:
unzip -q Fira_Code_v6.2.zip -d FiraCode

# Copiar .otf para fonts:
cp FiraCode/otf/*.otf "$HOME/.local/share/fonts/"

# Verificar:
ls "$HOME/.local/share/fonts/" | grep -i fira
```

---

## 2️⃣1️⃣ Atualizar Cache de Fontes do Sistema

Após instalar todas as fontes, atualize o cache:

```bash
# Atualizar cache de fontes:
fc-cache -fv "$HOME/.local/share/fonts"

# Aguarde a mensagem de conclusão
```

---

## 2️⃣2️⃣ Verificar Todas as Fontes Instaladas

```bash
# Listar todas as fontes no diretório:
ls "$HOME/.local/share/fonts/" | wc -l

# Listar apenas nossas fontes novas:
ls "$HOME/.local/share/fonts/" | grep -E "Inter|JetBrains|Cascadia|Fira"

# Verificar que o sistema as reconhece:
fc-list | grep -E "Inter|JetBrains|Cascadia|Fira"
```

Deve retornar as fontes instaladas.

---

## 2️⃣3️⃣ Usar as Fontes no VS Code

Abra VS Code e vá para `Ctrl+,` (Preferences):

Procure por "Font Family" e altere para uma das fontes instaladas:

```json
"editor.fontFamily": "JetBrains Mono",
// ou
"editor.fontFamily": "Cascadia Code",
// ou
"editor.fontFamily": "Fira Code",
```

Para interface GNOME, use **Inter** em:
- Settings → Appearance → Fonts

---

## 2️⃣4️⃣ Usar as Fontes no Terminal

Se usando **GNOME Terminal** ou **Tilix**:

1. Abra Preferences
2. Vá para "Text"
3. Clique em "Monospace" e selecione uma das fontes:
   - **JetBrains Mono Nerd Font** (recomendado)
   - **Cascadia Code**
   - **Fira Code**

---

## Limpeza de Arquivos Temporários

Após instalar todas as fontes, você pode remover os downloads:

```bash
# Remover arquivos temporários:
rm -rf /tmp/fonts

# Confirmar limpeza:
ls /tmp/fonts 2>/dev/null || echo "✓ Limpeza concluída"
```

---

# ✅ FINALIZAÇÃO

## Verificar Sistema

```bash
# Ver informações do sistema:
fastfetch

# Ver hostname:
hostnamectl

# Ver configuração git:
git config --global --list

# Verificar Codecs:
rpm -q ffmpeg

# Verificar VA-API:
rpm -q intel-media-driver

# Ver aplicativos instalados:
flatpak list --app
```

## Resumo do que foi feito

✅ Removidos Flatpaks pré-instalados  
✅ Removidos apps desnecessários do sistema base  
✅ Instalado RPM Fusion (Free + Non-Free)  
✅ Instalados Codecs FFmpeg completos  
✅ Instalado Intel VA-API (aceleração de vídeo)  
✅ Instalado VS Code  
✅ Instalado TLP (gerenciamento de bateria)  
✅ Configurado GNOME (dark mode, etc)  
✅ Configurado Git globalmente  
✅ Instalados aplicativos via Flatpak  
✅ Instalado eza (ls moderno)  

---

# 🔧 Troubleshooting

## Erro: "rpm-ostree: command not found"

Você está em um sistema que não é Silverblue. Use um sistema Fedora Silverblue.

## Erro: "sudo: command not found"

Silverblue não tem `sudo` por padrão. Use diretamente:

```bash
rpm-ostree install ...  # sem sudo
```

## Erro ao instalar Codecs: "already provided by"

Use `--allow-inactive`:

```bash
sudo rpm-ostree override remove ... --install --allow-inactive gstreamer1-plugin-libav
```

## VA-API não funciona após instalação

Verifique que reiniciou:

```bash
systemctl reboot
```

## eza não encontrado após instalação

Verifique que adicionou ao PATH:

```bash
echo $PATH | grep .local/bin
```

Se não aparecer, edite `~/.bashrc` ou `~/.zshrc`.

---

# 📝 Log de Execução

Anote aqui quando completar cada fase:

- [ ] Fase 1: Remoções (Data: _______)
- [ ] Fase 2: RPM Fusion + Reboot 1 (Data: _______)
- [ ] Fase 3: Codecs/VA-API + Reboot 2 (Data: _______)
- [ ] Fase 4: Pacotes RPM Overlay + Reboot 3 (Data: _______)
- [ ] Fase 5: GNOME Settings (Data: _______)
- [ ] Fase 6: Git Config (Data: _______)
- [ ] Fase 7: Flatpaks (Data: _______)
- [ ] Fase 8: Dev Tools (Data: _______)

---

**Status:** ✅ CONCLUÍDO

**Data de Conclusão:** _____________________

**Notas Adicionais:**
```
_________________________________________________________________
_________________________________________________________________
_________________________________________________________________
```

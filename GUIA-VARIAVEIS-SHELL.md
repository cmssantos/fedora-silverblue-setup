# 📝 Uso de Variáveis de Shell - Hostname e Git

## Atualização Realizada

O guia manual foi atualizado para usar **variáveis de shell** em duas seções:

1. ✅ **Seção 5️⃣ - Hostname**
2. ✅ **Seção 1️⃣3️⃣ - Git Config**

---

## 🎯 Por que Usar Variáveis?

### Vantagens

✅ **Reutilização**: Defina uma vez, use várias vezes  
✅ **Segurança**: Menos chance de erros de digitação  
✅ **Clareza**: Fica claro qual é o valor que será usado  
✅ **Manutenção**: Fácil mudar o valor em um lugar  
✅ **Documentação**: O valor aparece explicitamente  

### Exemplo Real

❌ **SEM variável** (difícil de entender):
```bash
hostnamectl set-hostname seu-hostname
git config --global user.name "Seu Nome"
git config --global user.email "seu.email@exemplo.com"
# ... 20 linhas depois ...
# Qual era o hostname mesmo? Qual o email?
```

✅ **COM variável** (claro e organizado):
```bash
MYHOSTNAME="seu-hostname"
GIT_NAME="Seu Nome"
GIT_EMAIL="seu.email@exemplo.com"

# Agora todas as operações usam as variáveis
hostnamectl set-hostname "$MYHOSTNAME"
git config --global user.name "$GIT_NAME"
git config --global user.email "$GIT_EMAIL"
```

---

## 🔧 Variável MYHOSTNAME

### Definição (Seção 5️⃣, Passo 1)

```bash
# Define o hostname desejado em uma variável
# SUBSTITUA 'seu-hostname' pelo seu hostname desejado
MYHOSTNAME="seu-hostname"

# Verificar a variável:
echo "Hostname será: $MYHOSTNAME"
```

### Uso Imediato (Seção 5️⃣, Passo 2)

```bash
# Usar a variável para definir o hostname:
hostnamectl set-hostname "$MYHOSTNAME"
```

### Verificação (Seção 5️⃣, Passo 3)

```bash
# Verificar a variável ainda existe:
echo "Hostname definido como: $MYHOSTNAME"

# Verificar no sistema:
hostnamectl
```

### Scope da Variável

```
┌─────────────────────────────────────────────┐
│ DEFINIÇÃO (Passo 1)                        │
│ MYHOSTNAME="seu-hostname"                  │
│ Variável existe a partir daqui ✅          │
└─────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────┐
│ USO (Passo 2)                              │
│ hostnamectl set-hostname "$MYHOSTNAME"     │
│ Variável ainda existe ✅                   │
└─────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────┐
│ VERIFICAÇÃO (Passo 3)                      │
│ echo "Hostname: $MYHOSTNAME"               │
│ Variável ainda existe ✅                   │
└─────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────┐
│ REBOOT 1 (Seção seguinte)                  │
│ systemctl reboot                           │
│ Variável se perde (terminal fecha) ❌      │
│ MAS hostname já foi salvo no sistema! ✅   │
└─────────────────────────────────────────────┘
```

---

## 🔧 Variáveis GIT_NAME e GIT_EMAIL

### Definição (Seção 1️⃣3️⃣, Passo 2)

```bash
# Define variáveis que serão usadas nesta sessão
# SUBSTITUA pelos seus dados:
GIT_NAME="Seu Nome Completo"
GIT_EMAIL="seu.email@exemplo.com"

# Verificar as variáveis:
echo "Nome Git: $GIT_NAME"
echo "Email Git: $GIT_EMAIL"
```

### Uso Imediato (Seção 1️⃣3️⃣, Passo 3)

```bash
# Usar as variáveis para configurar:
git config --global user.name "$GIT_NAME"
git config --global user.email "$GIT_EMAIL"

# Verificar que foram aplicadas:
echo "Configurado Git para: $GIT_NAME <$GIT_EMAIL>"
```

### Verificação (Seção 1️⃣3️⃣, Passo 6)

```bash
# Ver todas as configurações:
git config --global --list

# Ou verificar que usou a variável:
echo "Git Name: $(git config --global user.name)"
echo "Git Email: $(git config --global user.email)"
```

### Scope das Variáveis

```
┌─────────────────────────────────────────────┐
│ DEFINIÇÃO (Passo 2)                        │
│ GIT_NAME="Seu Nome"                        │
│ GIT_EMAIL="seu.email@exemplo.com"          │
│ Variáveis existem a partir daqui ✅        │
└─────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────┐
│ USO (Passo 3)                              │
│ git config --global user.name "$GIT_NAME"  │
│ git config ... user.email "$GIT_EMAIL"     │
│ Variáveis ainda existem ✅                 │
└─────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────┐
│ VERIFICAÇÃO (Passo 6)                      │
│ git config --global --list                 │
│ Variáveis podem ser verificadas ✅         │
│ (mas não aparecem no output, só valores)   │
└─────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────┐
│ PRÓXIMA SEÇÃO                              │
│ Variáveis se perdem (novo shell) ❌        │
│ MAS config git já foi salva! ✅            │
└─────────────────────────────────────────────┘
```

---

## ⚠️ Importante: Variáveis vs Configurações

### Variáveis Shell (Temporárias)

```bash
MYHOSTNAME="seu-hostname"
# Existe apenas nesta sessão do shell
# Se fechar terminal, desaparece ❌
# Se reiniciar, desaparece ❌
```

### Configurações do Sistema (Permanentes)

```bash
hostnamectl set-hostname "$MYHOSTNAME"
# Salva no sistema ✅
# Persiste após fechar terminal
# Persiste após reiniciar ✅
```

### Configurações Git (Permanentes)

```bash
git config --global user.name "$GIT_NAME"
# Salva em ~/.gitconfig ✅
# Persiste após fechar terminal
# Persiste após reiniciar ✅
```

---

## 🔄 Fluxo Completo da Seção de Hostname

```
PASSO 1: Definir variável
  MYHOSTNAME="seu-hostname"
  └─ Variável ✅ | Sistema ✗

PASSO 2: Aplicar ao sistema
  hostnamectl set-hostname "$MYHOSTNAME"
  └─ Variável ✅ | Sistema ✅

PASSO 3: Verificar
  echo "Hostname: $MYHOSTNAME"
  hostnamectl
  └─ Variável ✅ | Sistema ✅

REBOOT 1
  systemctl reboot
  └─ Variável ❌ (terminal fecha) | Sistema ✅ (salvo!)

APÓS REBOOT
  Variável perdida ❌
  MAS hostname persiste ✅
```

---

## 🔄 Fluxo Completo da Seção de Git

```
PASSO 2: Definir variáveis
  GIT_NAME="Seu Nome"
  GIT_EMAIL="seu.email@exemplo.com"
  └─ Variáveis ✅ | Git ✗

PASSO 3: Aplicar ao git
  git config --global user.name "$GIT_NAME"
  git config --global user.email "$GIT_EMAIL"
  └─ Variáveis ✅ | Git ✅

PASSO 4-6: Configurar + Verificar
  git config --global init.defaultBranch main
  git config --global --list
  └─ Variáveis ✅ | Git ✅

PRÓXIMA SEÇÃO
  Shell novo (ou mesmo shell continua)
  └─ Variáveis ❌ (ou ainda existem) | Git ✅ (salvo!)
```

---

## 💡 Dica: Reutilizar Variáveis na Mesma Seção

Se precisar usar a mesma variável várias vezes na **mesma seção**, antes de qualquer reboot:

```bash
# Defina uma vez no início
MYVARIABLE="valor-desejado"

# Use várias vezes:
comando1 "$MYVARIABLE"
comando2 "$MYVARIABLE"
comando3 "$MYVARIABLE"
echo "Finalmente, $MYVARIABLE foi aplicado!"
```

**Funciona porque:**
- ✅ Não há reboot entre os comandos
- ✅ Terminal continua aberto
- ✅ Variável permanece na memória

---

## 🎓 Exemplo Prático Completo

Imagine que você quer definir um hostname customizado:

### ❌ SEM variável (manual, propenso a erros)

```bash
# Passo 1: Definir
# (sem variável, precisa digitar o nome)
hostnamectl set-hostname meu-pc-dev

# Passo 2: Verificar
hostnamectl
# Saída: Static hostname: meu-pc-dev

# ... 10 linhas depois, você esquece qual era o hostname ...
```

### ✅ COM variável (claro e rastreável)

```bash
# Passo 1: Definir variável
MYHOSTNAME="meu-pc-dev"
echo "Hostname será: $MYHOSTNAME"

# Passo 2: Aplicar
hostnamectl set-hostname "$MYHOSTNAME"

# Passo 3: Verificar
echo "Hostname definido como: $MYHOSTNAME"
hostnamectl
# Saída: Static hostname: meu-pc-dev
# E você sabe que é o valor da variável!
```

---

## ✅ Checklist de Uso de Variáveis

### Hostname (Seção 5️⃣)
- [ ] Definiu `MYHOSTNAME="seu-hostname"`?
- [ ] Verificou com `echo "Hostname será: $MYHOSTNAME"`?
- [ ] Executou `hostnamectl set-hostname "$MYHOSTNAME"`?
- [ ] Verificou com `hostnamectl`?
- [ ] A variável aparece em `echo "Hostname: $MYHOSTNAME"`?

### Git (Seção 1️⃣3️⃣)
- [ ] Definiu `GIT_NAME="Seu Nome Completo"`?
- [ ] Definiu `GIT_EMAIL="seu.email@exemplo.com"`?
- [ ] Verificou com `echo "Nome: $GIT_NAME"` e `echo "Email: $GIT_EMAIL"`?
- [ ] Executou `git config --global user.name "$GIT_NAME"`?
- [ ] Executou `git config --global user.email "$GIT_EMAIL"`?
- [ ] Verificou com `git config --global --list`?

---

Status: ✅ VARIÁVEIS IMPLEMENTADAS

O guia agora usa variáveis para:
- ✅ Hostname (MYHOSTNAME)
- ✅ Git Name (GIT_NAME)
- ✅ Git Email (GIT_EMAIL)

Todas utilizadas na mesma seção sem reboots intermediários.

**IMPORTANTE:** Use SEUS dados pessoais ao definir as variáveis, não dados de exemplo!

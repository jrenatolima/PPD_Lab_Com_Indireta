readme_content = """# PPD Lab — Laboratório III — Comunicação Indireta com MQTT

Este repositório entrega a atividade do **Laboratório III** de Programação Paralela e Distribuída.
- **Atividade — `minerador.py`**: Sistema distribuído que implementa eleição de líder, coordenação e mineração (Proof of Work) utilizando o protocolo MQTT.

---

## 1) Pré-requisitos

- **Python 3.8+** (recomendado 3.10 ou superior).
- `pip` funcional.
- **Conexão com a Internet** (necessária para conectar ao broker público `broker.emqx.io`).
- Acesso de terminal (PowerShell no Windows; Terminal no macOS/Linux).

---

## 2) Estrutura do projeto

Trabalho_PPD/ ├─ minerador.py # Código fonte único (atua como Nó Minerador e Líder) ├─ Relatorio_Tecnico.pdf # Documentação da metodologia e testes └─ README.md # Este arquivo guia

```
Trabalho_PPD-CI/
├─ Relatorio_Tecnico.pdf          
├─ minerador.py
└─ README.md
```

---

## 3) Instalar dependências e preparar ambiente

**IMPORTANTE:** O projeto utiliza a biblioteca `paho-mqtt`.

> Recomenda-se usar **ambiente virtual** (venv) para isolar as dependências.

### 3.1 Windows (PowerShell)
```powershell
# entrar na pasta do projeto (onde está minerador.py)
cd C:\caminho\para\Trabalho_PPD

# criar e ativar venv (desbloqueando scripts somente nesta sessão)
py -m venv .venv
Set-ExecutionPolicy -ExecutionPolicy Bypass -Scope Process
.\.venv\Scripts\Activate.ps1

# instalar dependência
pip install paho-mqtt
```

### 3.2 macOS / Linux
```powershell
# entrar no diretório do projeto
cd /caminho/para/Trabalho_PPD

# criar e ativar venv
python3 -m venv .venv
source .venv/bin/activate

# instalar dependência
pip install paho-mqtt
```

### 4) Executar o Sistema (3 Nós)

Diferente do modelo Cliente-Servidor tradicional, este sistema é P2P (Peer-to-Peer) simétrico. Para funcionar corretamente, é necessário atingir o quórum mínimo de 3 nós.

Você precisará de 3 terminais abertos simultaneamente.

### 4.1 Terminal A (Nó 1)
```powershell
Mesmo terminal ja utilizado, basta apenas 

# 3. Rode o código
python minerador.py
```

### 4.2 Terminal B (Nó 2)
```powershell
# 1. Entre na pasta do projeto
cd C:\caminho\para\Trabalho_PPD

# 2. Ative o ambiente
Set-ExecutionPolicy -ExecutionPolicy Bypass -Scope Process
.\.venv\Scripts\Activate.ps1

# 3. Rode o código
python minerador.py
```

### 4.3 Terminal C (Nó 3)
```powershell
# 1. Entre na pasta do projeto
cd C:\caminho\para\Trabalho_PPD

# 2. Ative o ambiente
Set-ExecutionPolicy -ExecutionPolicy Bypass -Scope Process
.\.venv\Scripts\Activate.ps1

# 3. Rode o código
python minerador.py
```

#### Saída esperada:
```powershell
#>>> Rede sincronizada. Iniciando fase de votação...
#Líder determinado: XXXXX
```

## 5) Entendendo a Execução (Fluxo Detalhado)

O sistema opera como uma **Máquina de Estados Distribuída**. Abaixo descrevemos o ciclo de vida da aplicação após a inicialização dos 3 nós.

### 5.1 Fase de Sincronização (Init)
O sistema possui uma barreira de entrada (`barrier`). Enquanto o número de pares conectados for menor que 3, os nós permanecem em espera.
* **Log:** `Anunciando presença (X/3)...`
* **Ação:** Publicação periódica no tópico `sd/init`.

### 5.2 Fase de Eleição (Bully Simplificado)
Assim que o 3º nó entra, a barreira é liberada.
1. Todos os nós geram um `VoteID` aleatório.
2. Publicam seus votos em `sd/voting`.
3. Cada nó compara todos os votos recebidos. O maior `VoteID` (com desempate pelo `ClientID`) é eleito Líder.
* **Log:** `Líder determinado: [ID do Vencedor]`

### 5.3 Fase de Mineração (Running)
O sistema entra em loop contínuo de produção e processamento de tarefas.

#### A. O Líder (Gerador de Tarefas)
O nó eleito assume o papel de controlador. Ele cria uma transação e define a dificuldade (ex: hash deve começar com "000").
* **Log:** `--- [LIDER] Criando Transação T1 (Dificuldade 3) ---`
* **MQTT:** Publica em `sd/challenge`.

#### B. Os Mineradores (Worker Nodes)
Os nós não-líderes recebem o desafio e iniciam o *brute-force* (tentativa e erro) para achar o hash.
* **Log:** `Recebida Tarefa T1. Iniciando mineração...`
* **Log:** `Solução encontrada para T1: [Nonce]`
* **MQTT:** O primeiro a achar publica em `sd/solution`.

#### C. Validação e Consenso
O Líder recebe a solução, verifica o hash SHA-1 e declara o resultado.
* **Log:** `>>> T1 FINALIZADA. Vencedor: [ID do Minerador]`
* **MQTT:** Publica em `sd/result`.
* **Ação:** O ciclo reinicia imediatamente com a Transação T2."
# Conectando o Garmin ao Claude Desktop (Windows) — Guia Completo

Este guia ensina, do zero, como conectar sua conta do Garmin Connect ao Claude Desktop, permitindo que o Claude leia seus treinos, frequência cardíaca, sono e outras métricas diretamente do Garmin, sem precisar enviar prints manualmente.

**Importante:** esse tipo de conexão (MCP local) só funciona enquanto o Claude Desktop estiver aberto no computador em que foi configurado. Ele não sincroniza para o celular nem funciona com o PC desligado.

**Tempo estimado:** ~20-30 minutos
**Sistema:** Windows 10 ou 11

---

## Pré-requisitos

- Ter o **Claude Desktop** instalado ([claude.ai/download](https://claude.ai/download))
- Ter uma conta ativa no **Garmin Connect**
- (Opcional) Criar um projeto no Claude Desktop chamado, por exemplo, "Treinador", com instruções personalizadas de como você quer ser ajudado

---

## Parte 1 — Instalar Python e Git

### 1.1 Instalar o Python 3.11

1. Acesse: https://www.python.org/downloads/windows/
2. Baixe o instalador **Windows installer (64-bit)** da versão **3.11.x**
3. Abra o instalador
4. **Importante:** na primeira tela, marque a caixa **"Add Python to PATH"** antes de continuar
5. Clique em **Install Now** e aguarde terminar

### 1.2 Verificar a instalação

1. Aperte a tecla `Windows`, digite `cmd`
2. Clique com o botão direito em "Prompt de Comando" → **Executar como administrador**
3. Digite:
   ```
   python --version
   ```
4. Deve aparecer `Python 3.11.x`. Se der erro de "comando não reconhecido", reinstale marcando a caixa do PATH.

> Se você já tiver uma versão diferente do Python instalada (ex: 3.14), não tem problema — dá para ter as duas ao mesmo tempo. Use `py -3.11 --version` para chamar especificamente a 3.11 quando precisar.

### 1.3 Instalar o Git

1. Acesse: https://git-scm.com/download/win (o download começa automaticamente)
2. Abra o instalador e vá clicando **Next** mantendo as opções padrão
3. Finalize com **Install** → **Finish**
4. Verifique no Prompt de Comando:
   ```
   git --version
   ```
   Deve aparecer algo como `git version 2.x.x`

---

## Parte 2 — Instalar o conector Garmin (garmin_mcp)

O projeto usado é o **garmin_mcp**, de código aberto, disponível em:
https://github.com/Taxuspt/garmin_mcp

### 2.1 Criar a pasta do projeto

No Prompt de Comando (como administrador), rode um comando por vez:

```
cd %USERPROFILE%
mkdir garmin_mcp_server
cd garmin_mcp_server
```

### 2.2 Baixar o projeto

```
git clone https://github.com/Taxuspt/garmin_mcp.git .
```

(Não esqueça do ponto `.` no final — ele baixa o conteúdo direto na pasta atual)

### 2.3 Criar o ambiente virtual Python

```
python -m venv venv
venv\Scripts\activate
```

O terminal deve passar a mostrar `(venv)` no começo da linha — isso indica que o ambiente isolado está ativo.

### 2.4 Instalar as dependências

```
python -m pip install --upgrade pip
pip install -e .
```

Aguarde até aparecer `Successfully installed garmin-mcp...`

---

## Parte 3 — Autenticar com sua conta Garmin

Ainda no mesmo Prompt de Comando (com `(venv)` ativo), rode:

```
garmin-mcp-auth
```

- Digite seu e-mail do Garmin Connect
- Digite sua senha (não aparece nada na tela ao digitar, é normal)
- Se você tiver autenticação de dois fatores (MFA) ativada, será solicitado o código

Quando aparecer `Authentication successful!`, os tokens de acesso foram salvos localmente no seu PC (pasta `~/.garminconnect`) e valem por aproximadamente **6 meses**.

---

## Parte 4 — Configurar o Claude Desktop

### 4.1 Descobrir seu usuário do Windows

```
echo %USERNAME%
```

Anote o nome que aparecer.

### 4.2 Descobrir o caminho exato do executável

Com `(venv)` ainda ativo:

```
where garmin-mcp
```

Vai aparecer um caminho parecido com:
```
C:\Users\SEUNOME\garmin_mcp_server\venv\Scripts\garmin-mcp.exe
```

Anote esse caminho completo.

### 4.3 Localizar o arquivo de configuração do Claude

1. Aperte `Windows + R`
2. Cole: `%APPDATA%\Claude` e aperte Enter

> **Se essa pasta não abrir** (erro "Windows não pode encontrar..."), pode ser que seu Claude Desktop tenha sido instalado via Microsoft Store. Nesse caso, dentro do próprio app, vá em **Configurações → Desenvolvedor → Editar Config** — isso cria/abre o arquivo automaticamente no caminho correto (geralmente algo como `AppData\Local\Packages\Claude_xxxxx\LocalCache\Roaming\Claude`).

3. Localize o arquivo `claude_desktop_config.json`
4. Clique com o botão direito → Abrir com → **Bloco de Notas**

### 4.4 Editar o arquivo (sem apagar o que já existe)

⚠️ **Atenção:** se o arquivo já tiver conteúdo (outras configurações do Claude Desktop, como preferências salvas), **não apague tudo** — isso pode quebrar outras funções do app. Em vez disso, insira o bloco `mcpServers` logo após a primeira chave `{` do arquivo, adicionando uma vírgula depois para separar do restante.

**Se o arquivo estiver vazio ou for novo**, use exatamente isto (substituindo `SEUNOME` pelo valor real, em 2 lugares):

```json
{
  "mcpServers": {
    "garmin-local": {
      "command": "C:\\Users\\SEUNOME\\garmin_mcp_server\\venv\\Scripts\\garmin-mcp.exe",
      "env": {
        "GARMIN_COOKIES_FILE": "C:\\Users\\SEUNOME\\.garminconnect\\cookies.json"
      }
    }
  }
}
```

**Se o arquivo já tiver conteúdo**, insira assim (exemplo com outras chaves já existentes):

```json
{
  "mcpServers": {
    "garmin-local": {
      "command": "C:\\Users\\SEUNOME\\garmin_mcp_server\\venv\\Scripts\\garmin-mcp.exe",
      "env": {
        "GARMIN_COOKIES_FILE": "C:\\Users\\SEUNOME\\.garminconnect\\cookies.json"
      }
    }
  },
  "outraChaveQueJaExistia": "valor",
  "maisConfiguracoes": { }
}
```

**Pontos críticos:**
- Use sempre barra dupla `\\` nos caminhos do Windows
- Substitua `SEUNOME` em **ambos** os lugares que ele aparece
- Não esqueça da vírgula depois do bloco `mcpServers` se houver mais conteúdo depois

### 4.5 Salvar

`Ctrl + S`. Se aparecer diálogo pedindo tipo de arquivo, escolha "Todos os Arquivos (*.*)" e mantenha o nome terminando em `.json` (não deixe virar `.json.txt`).

### 4.6 Reiniciar o Claude Desktop — **passo mais importante e mais esquecido**

Fechar só a janela (clicar no X) **não é suficiente** — o app continua rodando em segundo plano e não recarrega a configuração nova.

1. Vá na bandeja do sistema (ícones perto do relógio, no canto inferior direito da tela)
2. Clique com o botão direito no ícone do Claude
3. Escolha **Sair / Quit**
4. Abra o Claude Desktop de novo pelo Menu Iniciar

---

## Parte 5 — Testar

Abra uma conversa (de preferência dentro do projeto "Treinador") e pergunte:

> "Quais foram meus últimos 5 treinos?"

Se o Claude responder com dados reais e específicos do seu Garmin (datas, distância, pace, FC), funcionou. 🎉

---

## Solução de problemas comuns

| Problema | Causa provável | Solução |
|---|---|---|
| "Server disconnected" | Barra simples `\` em vez de `\\` no JSON | Revisar todos os caminhos no arquivo |
| Não reconhece dados novos | Claude Desktop não foi reiniciado de verdade | Sair pela bandeja do sistema, não só fechar a janela |
| "python não é reconhecido" | Não marcou "Add Python to PATH" | Reinstalar o Python marcando a caixa |
| "garmin-mcp não encontrado" | Ambiente virtual não ativado | Rodar `venv\Scripts\activate` na pasta do projeto antes dos comandos |
| Pasta `%APPDATA%\Claude` não existe | Instalação via Microsoft Store usa outro caminho | Usar "Editar Config" dentro de Configurações → Desenvolvedor no app |
| Token expirou (~6 meses) | Validade normal do login Garmin | Rodar `garmin-mcp-auth` novamente com o ambiente virtual ativo |
| Funciona no PC mas não no celular | MCP local só roda na máquina configurada | Não sincroniza por padrão; exigiria hospedar o servidor remotamente (fora do escopo deste guia) |

---

## Segurança

- As credenciais do Garmin não ficam salvas em texto puro pelo app — o processo de autenticação gera tokens OAuth armazenados localmente em `~/.garminconnect`
- O servidor roda **localmente**, sem enviar dados para terceiros
- Para revogar o acesso, basta apagar as pastas `garmin_mcp_server` e `.garminconnect`, e remover o bloco `mcpServers` do arquivo de configuração

---

## Créditos

Servidor MCP utilizado: [Taxuspt/garmin_mcp](https://github.com/Taxuspt/garmin_mcp) (projeto de código aberto).

# LLM_RE_Local

Comando `llm` para conversar no terminal do Windows com um modelo de linguagem próprio, rodando no Google Colab (GPU T4) ou no PC (CPU), com fallback automático entre os dois. Só biblioteca padrão do Python 3.13.

**Nome:** LLM + RE (*Reverse Engineering*, engenharia reversa) + Local. Uma LLM que roda na minha máquina (ou no meu Colab), feita para engenharia reversa.

## Propósito

Ter uma LLM sob meu controle, rodando no meu PC ou no meu próprio Colab, para apoiar a análise de firmware, formatos e protocolos de eletrônicos que já tenho em mãos: televisores, Amazon Alexa, multimídias chinesas, dispositivos Google, celulares antigos e eletrônicos em geral. O foco é privacidade: firmware, dumps e logs dos meus aparelhos ficam na minha máquina ou no meu Colab, sem passar por serviços de terceiros. Serve para analisar dumps, logs e código e para explicar formatos e protocolos.

O modelo usado é o `DeepSeek-V4-Pro-Qwen3.5-9B-UNCENSORED-heretic` (GGUF, via llama.cpp).

## Como foi feito

- **Cliente `llm`** (`llm/`): chat de terminal com streaming, histórico preservado e escolha do backend (`auto`, `local` ou Colab). Fala com o `llama-server` pela API compatível com OpenAI.
- **Local**: `scripts/instalar_local.ps1` baixa o llama.cpp (CPU) e o modelo Q4_K_M (5,63 GB) para `%USERPROFILE%\.llm`.
- **Colab**: `colab/llm_colab.ipynb` sobe o mesmo servidor com GPU T4 (modelo Q6_K) e, opcionalmente, um túnel cloudflared protegido por chave.
- **Fallback**: se o Colab cair no meio da conversa, o `llm` sobe o servidor local e reenvia o mesmo histórico.
- **Arquivos**: `/ler`, `/prompt` e `/salvar` só acessam uma pasta definida no config; a IA nunca mexe em arquivos sozinha.
- **Testes**: `unittest`, sem rede.

## Níveis

| Nível | Onde roda | Como usa | Velocidade medida | Risco |
|---|---|---|---|---|
| N0 | PC (CPU, i5-1135G7, 16 GB) | `llm` no terminal | ~2,4 tok/s | nenhum |
| N1 | Colab GPU T4 | chat dentro do notebook | ~27 tok/s | dentro das regras do Colab |
| N2 | Colab GPU T4 | `llm` no terminal via túnel | ~27 tok/s | expõe o servidor do Colab por túnel; confira os termos de uso do seu plano do Colab antes de usar e desconecte ao terminar |

A medição do N0 foi feita com pouca RAM livre. Não há keep-alive do Colab nem ocultação de tráfego, de propósito. Detalhes do hardware em `docs/ANALISE_HARDWARE.md`.

## Como usar

**Local (N0)**
1. Tenha ~7 GB livres em disco e RAM livre.
2. `powershell -ExecutionPolicy Bypass -File scripts\instalar_local.ps1` (pode repetir; retoma o download).
3. `.\llm.cmd status` e depois `.\llm.cmd` (ou ponha a pasta no PATH e use só `llm`).

**Colab (N1 e N2)**
1. Abra `colab/llm_colab.ipynb` no Colab, com ambiente de execução GPU T4.
2. Execute as células na ordem: 1 (binário CUDA), 2 (modelo e servidor), 3 (N1: chat no notebook).
3. N2: execute a célula 4, copie a linha `llm colab <url> <chave>` e cole no terminal do Windows.
4. Ao terminar: células 5 e 6 e **Desconectar e excluir ambiente de execução**.

Config e chave ficam em `%USERPROFILE%\.llm\config.json`, fora do repositório.

## Comandos

| Comando | O que faz |
|---|---|
| `llm` / `llm chat` | chat (Colab se configurado e respondendo, senão local) |
| `llm chat --sem-pensar` | desliga o thinking (mais rápido) |
| `llm chat --local` | força o local |
| `llm status` | mostra o backend e se Colab/local respondem |
| `llm colab <url> <chave>` / `--limpar` | salva ou remove o Colab |
| `llm local` / `llm auto` | fixa o backend em local / volta ao automático |
| `llm parar` | para o servidor local |

No chat: `/sair`, `/limpar`, `/backend`, `/pensar`, `/ler [arquivo ou pasta]`, `/prompt <arquivo>`, `/salvar <nome.txt>`.

## Solução de problemas

- **Servidor local não sobe:** feche navegador e outros programas; veja `%USERPROFILE%\.llm\llama-server.log`.
- **Porta 8080 ocupada:** mude `porta_local` no `config.json` ou rode `llm parar`.
- **Binário CUDA falhou no Colab:** use a célula 1b (compila para a arquitetura 75, ~10-15 min).

## Desenvolvimento

`python -m unittest discover -s tests`

## Uso responsável

Use só em equipamentos seus ou com autorização. O projeto não inclui ferramentas que agem sozinhas sobre dispositivos nem sobre arquivos.

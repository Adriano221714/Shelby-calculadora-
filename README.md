# SHELBY CALCULATOR — versão publicável

Calculadora (básica, científica, financeira, conversões, estatística, equações, cálculo) + assistente de IA.
Os cálculos correm no dispositivo do utilizador (funcionam sem Internet). O assistente usa um pequeno servidor
que guarda a sua chave da API — a chave **nunca** vai para o navegador.

## Estrutura

    index.html                      página completa (HTML, CSS e JavaScript)
    netlify/functions/chat.mjs      servidor do assistente (pedido à API da Anthropic)
    netlify.toml                    configuração do Netlify
    README.md                       estas instruções

## O que precisa

1. Uma conta no **Netlify** (tem plano gratuito) e uma conta no **GitHub**.
2. Uma **chave de API da Anthropic**, criada na Anthropic Console (console.anthropic.com).
   A API é paga por utilização e o custo fica na sua conta.
   Na Console defina um **limite de gastos mensal** antes de divulgar a ligação.

## Publicar (passo a passo)

1. No GitHub crie um repositório novo e carregue estes ficheiros, mantendo as pastas como acima.
2. No Netlify escolha adicionar um site novo a partir de um repositório Git e seleccione esse repositório.
   Não precisa de comando de compilação (deixe em branco); a pasta a publicar é a raiz ("."), já definida no `netlify.toml`.
3. Nas definições do site, em variáveis de ambiente, crie:
   - `ANTHROPIC_API_KEY` = a sua chave (obrigatória)
   - `ANTHROPIC_MODEL` = nome do modelo (opcional; por defeito `claude-sonnet-5-5`)
   - `RATE_LIMIT_PER_MIN` = pedidos por minuto e por IP (opcional; por defeito 20)
4. Faça um novo deploy para as variáveis ficarem activas e abra a ligação que o Netlify lhe der.

Os nomes exactos dos menus do Netlify e do GitHub podem mudar; em caso de dúvida consulte a documentação deles.
Pode usar outro alojamento com funções serverless (por exemplo Cloudflare Pages ou Vercel), mas terá de adaptar
`chat.mjs` ao formato desse serviço.

## Testar no seu computador (opcional)

Requer Node.js. Na pasta do projecto:

    npm install -g netlify-cli
    ANTHROPIC_API_KEY=a_sua_chave netlify dev

Abra o endereço indicado (normalmente http://localhost:8888).
Se abrir o `index.html` directamente (ficheiro), a calculadora funciona mas o assistente não.

## Como funciona o assistente

- O navegador envia a conversa para `/api/chat`; o servidor junta a chave e chama a API.
- Quando a IA precisa de um resultado numérico, pede a ferramenta «calcular»; o navegador resolve-a com o motor
  matemático local e devolve o valor, e a IA escreve a explicação.
- Para trocar de fornecedor de IA, altere apenas `chat.mjs` (URL, cabeçalhos e formato do pedido) e a pequena
  parte de `index.html` que lê a resposta (função `ask`).

## Segurança e privacidade

- A chave está só na variável de ambiente do servidor. O servidor valida o pedido (papéis, tamanho, número de
  mensagens) e devolve erros genéricos, sem detalhes internos.
- O limite por IP é simples e por instância do servidor: reduz abusos ligeiros, mas não os impede.
  O limite de gastos na Anthropic Console é a protecção real contra custos inesperados.
  Para uso público alargado, acrescente autenticação.
- O histórico da calculadora fica só no dispositivo de cada pessoa (armazenamento local do navegador).
- O texto escrito no assistente é enviado ao serviço de IA; a página avisa disto.

## Testes realizados (no desenvolvimento)

- Motor matemático: 23 casos automáticos passaram (precedência, parêntesis, negativos, decimais, percentagens,
  factorial, trigonometria em graus, divisão por zero, expressão incompleta, símbolos inválidos, números muito grandes).
- Ferramentas: estatística, equações de 1.º e 2.º grau (incluindo raízes complexas), sistemas lineares, determinante,
  derivada, integral, limites (incluindo laterais e infinito), conversões, juros, prestação, margem, sementes,
  densidade e produtividade — verificados com casos de resultado conhecido.
- Servidor: verificados 405 (método), 503 sem chave, 400 (corpo inválido, papel inválido, pedido demasiado grande),
  envio correcto ao fornecedor com um servidor simulado, e tratamento do erro do fornecedor.

## Ainda não testado / limitações

- **Não foi feita nenhuma chamada real à API da Anthropic nem um deploy real**: só depois de configurar a chave
  vai poder confirmar que o assistente responde. Faça esse teste antes de divulgar.
- Não testado em iPhone, Android e outros navegadores, nem com leitores de ecrã. Navegação por teclado básica.
- O assistente não mostra a resposta a ser escrita (aparece completa no fim) e não tem tabelas/fórmulas formatadas.
- Derivada, integral e limite são numéricos e aproximados.
- Ainda não existe: fracções, regra de três, conversão de moedas, favoritos, pesquisa no histórico, exportação,
  anexos de documentos/imagens, contas de utilizador e sincronização.

# OSUK — site e Pix na Vercel

Este pacote contém o site atual, o servidor para criar cobranças Pix, o banco de pedidos e a confirmação por webhook da GoatPay. Foi testado localmente com respostas simuladas, sem criar cobranças. A conexão real ainda depende das configurações nas suas contas. Use Chrome, Edge ou Safari habitual para fazer essas configurações; não envie senhas ou chaves pelo chat.

## 1. Atualizar o GitHub

Extraia **OSUK-Pix-Vercel.zip**. Envie o conteúdo da pasta `osuk-vercel` para a raiz do repositório conectado ao seu projeto da Vercel. No GitHub use **Add file → Upload files**, ou sincronize a pasta pelo GitHub Desktop. Preserve a estrutura:

```text
public/            site, imagens e scripts públicos
api/               rotas que a Vercel executa no servidor
osuk-server/       regras, banco e catálogo de entrega
package.json
vercel.json
.gitignore
.env.example
README.md
```

Use um repositório **Private**, pois `osuk-server/fulfillment-catalog.json` contém os links de entrega. Nunca mova `osuk-server` para `public`. Não envie o ZIP sozinho nem uma pasta extra envolvendo essa estrutura. Guarde sua versão anterior para poder voltar a ela.

Na Vercel, mantenha **Framework Preset: Other**, **Root Directory: raiz do repositório**, **Output Directory: public** e **Build Command: vazio**. Deixe a instalação padrão das dependências habilitada; `pg` será instalado por `package.json`. O arquivo `vercel.json` já inclui as rotas do servidor. Se tiver escolhido outra Root Directory antes, ajuste-a para a pasta que contém `package.json`, `public` e `api`.

## 2. Criar o banco de pedidos

No painel da Vercel, abra seu projeto → **Storage**, escolha uma integração de **PostgreSQL** disponível ou use um PostgreSQL que já possua. Escolha o plano conscientemente antes de contratar. Copie a URL de conexão fornecida pelo banco, preferindo a conexão com pooling e TLS para funções serverless. Não use uma chave pública de navegador no lugar dessa URL.

O servidor cria as tabelas `osuk_orders`, `osuk_webhook_events` e `osuk_rate_limits` na primeira solicitação que acessa o banco. A conta da conexão precisa ter permissão para criar essas tabelas e seus índices. Se sua conta não permitir criação, execute o esquema exportado em `osuk-server/database.cjs` por uma conta administrativa e mantenha as permissões de leitura/escrita para a aplicação.

## 3. Cadastrar as variáveis na Vercel

No seu navegador normal: projeto → **Settings → Environment Variables**. Cadastre em **Production**:

| Nome | Valor |
| --- | --- |
| `APP_URL` | `https://osukotimizers.vercel.app` — sem `/pagamento.html` |
| `DATABASE_URL` | URL privada de conexão do PostgreSQL |
| `GOATPAY_API_KEY` | Chave da sua conta, obtida no painel GoatPay em Integrações → Chaves de API |
| `GOATPAY_WEBHOOK_SECRET` | Segredo `whsec_...` do webhook criado no próximo passo |
| `ORDER_TOKEN_SECRET` | Segredo aleatório exclusivo, com pelo menos 32 caracteres; recomendado: 64 caracteres hexadecimais |

Gere o último segredo com o gerador do seu gerenciador de senhas, usando pelo menos 64 caracteres aleatórios e evitando palavras/frases. Guarde-o com segurança e mantenha-o estável: mudar esse segredo invalida a recuperação automática de tentativas antigas. Não preencha segredos no arquivo `public/config.js`, no GitHub nem no `.env.example`.

Se mudar o domínio, ajuste `APP_URL`. O servidor aceita a criação da compra somente a partir desse domínio. Não compartilhe chaves de produção com deployments de Preview; para testar previews, use credenciais e banco separados quando disponíveis. Sem essas variáveis, o servidor retorna pagamento não configurado.

## 4. Criar a confirmação automática na GoatPay

No painel da GoatPay, em **Integrações → Webhooks** (os nomes podem variar), cadastre um endpoint HTTPS para:

```text
https://osukotimizers.vercel.app/api/webhooks/goatpay
```

Selecione estes eventos:

```text
payment.paid
payment.failed
payment.pix.expired
refund.completed
payment.refunded
```

Copie o segredo do endpoint para `GOATPAY_WEBHOOK_SECRET` na Vercel. O segredo do webhook é diferente da chave da API. Se o webhook estiver associado a uma chave específica, use a mesma chave cadastrada em `GOATPAY_API_KEY` para que as cobranças sejam notificadas. Se o painel não oferecer essa criação, consulte o suporte ou o endpoint de criação documentado; não tente criar uma confirmação usando a URL de retorno do comprador.

O servidor verifica a assinatura sobre os bytes originais, confere o ID do pedido, o valor em reais, o tipo de pagamento, a moeda e o ID da cobrança. Eventos repetidos são tratados uma única vez. O retorno do visitante não aprova uma compra. Reembolso confirmado bloqueia novas consultas aos links da entrega.

## 5. Publicar e conferir

Após salvar as variáveis, abra **Deployments → Redeploy** na Vercel. Variáveis novas só passam a valer em um novo deploy. Abra:

```text
https://osukotimizers.vercel.app/api/health
```

`configuration_required` lista os nomes que faltam. `configuration_present` confirma apenas que os campos estão preenchidos; **não comprova** que a chave é válida, que o banco está acessível ou que o webhook entrega eventos.

No painel GoatPay, use o teste de envio do webhook, se disponível, e confira resposta HTTP 200. Esse teste verifica a assinatura e o acesso ao endpoint; não libera um pack. Confira os logs da Vercel se houver falha. A documentação consultada oferece produção para a API Pix; não foi presumido um ambiente de sandbox. Faça a verificação final de uma compra real você mesmo, consciente de que pode haver cobrança e tarifa.

O fluxo esperado é: escolher pack → informar e confirmar e-mail → gerar QR Code/copia e cola → pagar → confirmação assinada → foguete de aprovação → **Acessar pack** → chat com tutorial e download do produto comprado. O preço vem do servidor: Ultra R$ 35,90; Valorant R$ 22,90; Completo R$ 17,90. A página consulta a confirmação a cada 15 segundos por até 10 minutos; depois use **Atualizar pagamento**. O código Pix vence em 30 minutos. O acesso ao pedido dura 30 dias e fica nesta sessão do navegador.

O e-mail identifica o pedido para suporte. **Ainda não há serviço de envio de e-mails ou recuperação por e-mail integrado.** O pack e o tutorial são entregues no chat após aprovação. Para enviar também por e-mail, será necessária outra integração. Os links de Drive/YouTube já fornecidos estão no catálogo privado. Depois de recebidos, esses links podem ser compartilhados por quem tem acesso; este pacote não transforma arquivos do Drive em downloads com autenticação própria.

## Erros comuns

| Sinal | Onde conferir |
| --- | --- |
| `/api/orders/current` retorna 404 | Verifique `api` na raiz, Root Directory e novo deploy |
| Pagamento não configurado | Confira as cinco variáveis de Production e redeploy |
| Falha ao criar Pix | Validade/permissões da chave, saldo/condições da conta e logs da Vercel |
| Erro no banco | URL correta, TLS, pooling e permissão das tabelas |
| Pagou, mas não liberou | Entregas do webhook, segredo correto, chave vinculada e `payment.paid` |
| Pedido iniciado precisa ser conferido | Resultado incerto de uma solicitação: confirme no painel/Discord antes de criar outra cobrança |

Uma tentativa com resposta incerta não gera automaticamente outra cobrança. Isso evita cobrança duplicada, mas pode exigir conferência pelo vendedor no painel e no banco. O token de acesso é privado: não divulgue a URL com `#acesso=`. Para alterar preços ou campanhas, mantenha `osuk-server/payment-rules.cjs` e o catálogo público sincronizados. O desconto de demonstração não muda o preço do servidor.

## Fontes oficiais

- [Criar Pix na GoatPay](https://docs.goatpay.com.br/api-reference/endpoint/payment-pix/create)
- [Webhooks e assinatura GoatPay](https://docs.goatpay.com.br/api-reference/guides/webhooks)
- [Funções Node.js na Vercel](https://vercel.com/docs/functions/runtimes/node-js)
- [Variáveis de ambiente na Vercel](https://vercel.com/docs/environment-variables)
- [Criar repositório GitHub](https://docs.github.com/en/repositories/creating-and-managing-repositories/creating-a-new-repository)

# Publicar OSUK no GitHub e na Vercel, depois conectar o Pix

Conferido em 07/10/2026.

O visual, as páginas de packs, o formulário de e-mail, a consulta de status, a animação de aprovação e o chat de entrega estão prontos. Ainda falta implementar o servidor de pagamento, o banco de pedidos, o webhook e o envio de e-mail. Uma chave de API cadastrada na Vercel não implementa esses componentes.

## 1. Extrair o pacote preparado

Baixe OSUK-GitHub-Vercel.zip e use Extrair tudo no Windows. Dentro da pasta osuk-github-vercel você encontrará:

```text
public/
  index.html
  ultra-pack.html
  valorant-booster.html
  pack-completo.html
  pagamento.html
  entrega.html
  config.js
  ... outros estilos, scripts e páginas
  assets/
vercel.json
.gitignore
README.md
```

Esse pacote publica somente o site. Os links privados dos produtos não foram incluídos. Guarde também o pacote completo original: a pasta osuk-server será utilizada na implementação do servidor, fora de public.

## 2. Enviar ao GitHub

1. Entre na sua conta do GitHub.
2. Clique no botão + e depois em New repository.
3. Use o nome osuk-optimizers.
4. Selecione Private e clique em Create repository.
5. Na página do repositório, escolha uploading an existing file. Se essa opção não aparecer, use Add file → Upload files.
6. Arraste public, vercel.json, .gitignore e README.md para o envio. Arraste o conteúdo de osuk-github-vercel; não envie somente o ZIP.
7. Preserve todas as subpastas de assets.
8. Escreva Site inicial OSUK e clique em Commit changes.
9. Confira que public e vercel.json estão na raiz do repositório.

Não envie a pasta geral da conversa, work, screenshots, .env ou credenciais. Para atualizações frequentes, o GitHub Desktop também pode sincronizar a pasta extraída.

Referência: [criar repositório](https://docs.github.com/en/repositories/creating-and-managing-repositories/creating-a-new-repository).

## 3. Publicar na Vercel

1. Entre na Vercel e conecte sua conta do GitHub.
2. Escolha Add New → Project.
3. Importe o repositório osuk-optimizers. Se ele não aparecer, ajuste o acesso da integração do GitHub para esse repositório.
4. Confira as configurações abaixo e clique em Deploy.

| Campo | Valor deste pacote |
| --- | --- |
| Framework Preset | Other |
| Root Directory | Raiz do repositório, sem selecionar public |
| Build Command | Vazio; habilite Override se necessário |
| Output Directory | public |
| Install Command | Sem comando personalizado |

O vercel.json já define a saída public e dispensa compilação. Quando o deploy terminar, abra o endereço HTTPS atribuído pela Vercel. O localhost 127.0.0.1 funciona somente no seu computador e não deve ser usado no webhook.

Referências: [importar GitHub](https://vercel.com/docs/git/vercel-for-github) e [configurar projeto estático](https://vercel.com/docs/builds/configure-a-build).

Como é uma loja comercial, use um plano que permita esse uso. O Hobby é limitado a uso pessoal não comercial. [Regras do Hobby](https://vercel.com/docs/plans/hobby).

## 4. Revisar a publicação

Abra a home, os três packs, pagamento.html e entrega.html. Confira logo, ícones, imagens, carrossel, menu mobile, Discord, preços e benefícios. O botão de pagamento continuará indisponível até conectar o servidor; isso é esperado.

No public/config.js, preencha siteUrl com o endereço definitivo HTTPS do site. Não coloque chave de API nesse arquivo.

Para domínio próprio, abra Settings → Domains no projeto, adicione o domínio e aplique exatamente os registros DNS mostrados pela Vercel. Depois use esse domínio em siteUrl e na configuração de retorno/webhook. O endereço *.vercel.app também serve para a publicação inicial.

Revise as condições comerciais pendentes, os preços anteriores de exemplo, a promoção e a política de privacidade antes de abrir a venda.

## 5. Implementar o servidor de pagamentos

Esta etapa exige desenvolvimento; esses endpoints ainda não existem no pacote. No mesmo projeto da Vercel, o código das funções ficará em api, na raiz, e os módulos privados fora de public. Será necessário um banco de dados persistente para pedidos, estados, tokens e eventos processados; arquivos temporários e memória da função não substituem o banco.

| Rota da OSUK a implementar | Função |
| --- | --- |
| POST /api/orders | Validar e-mail e pack, definir preço no servidor, criar cobrança/checkout e salvar pedido |
| GET /api/orders/current | Autenticar token e retornar o estado do pedido; liberar os links apenas após aprovação |
| POST /api/webhooks/goatpay | Receber e validar as notificações do provedor |

As rotas acima pertencem à OSUK. Não são as rotas da API da GoatPay. O contrato completo já está no arquivo INTEGRACAO-PAGAMENTO.md do pacote original.

O frontend atual espera uma checkoutUrl HTTPS e um accessToken ao criar o pedido. O adaptador deve fornecer um checkout hospedado compatível. Se a integração escolhida retornar apenas QR Code/Pix copia e cola, será necessário adaptar a página de pagamento para exibi-los; adicionar a chave não faz essa adaptação.

O servidor deve registrar uma referência única por pedido, calcular os preços de Ultra (3590 centavos), Valorant (2290) e Completo (1790), evitar cobranças duplicadas e conferir valor/moeda/referência antes de liberar conteúdo. O catálogo de entrega fornecido permanece privado, fora de public.

Para receber o acesso por e-mail, configure também um serviço de envio, remetente autorizado e recuperação de acesso. O e-mail preenchido no formulário sozinho não envia mensagens.

## 6. Cadastrar a chave e demais configurações

Na conta GoatPay, localize a área de API/integrações e gere a chave com as permissões necessárias. Confira o painel e a documentação da sua conta; os nomes dos menus podem variar. A documentação oficial consultada utiliza X-API-Key nas chamadas feitas pelo servidor. [Integração GoatPay](https://docs.goatpay.com.br/pages/guides/integracao-ia).

Na Vercel, abra o projeto → Settings → Environment Variables e cadastre as variáveis. Estes nomes são uma sugestão para o servidor que será implementado; o código precisará lê-los explicitamente.

| Nome sugerido | Valor |
| --- | --- |
| GOATPAY_API_KEY | Chave privada da conta GoatPay |
| GOATPAY_WEBHOOK_SECRET | Segredo usado para validar o webhook |
| DATABASE_URL | Conexão privada do banco de pedidos |
| APP_URL | Endereço HTTPS definitivo da OSUK |
| EMAIL_API_KEY | Credencial do serviço de e-mail escolhido, se exigida por ele |

Selecione Production para credenciais de produção e use Secret para credenciais quando a interface oferecer essa opção. Não habilite pagamentos reais em previews automaticamente. Salve e faça um novo deployment para as funções receberem as alterações. Essas variáveis não são injetadas automaticamente em arquivos HTML/JS estáticos. [Variáveis da Vercel](https://vercel.com/docs/environment-variables).

Não cole a chave no chat, no GitHub ou em public/config.js. Se uma chave real já tiver sido publicada, revogue-a na conta e gere outra.

## 7. Configurar confirmação e conectar o site

Depois de implementar e publicar a função, cadastre no provedor a URL HTTPS https://SEU-DOMINIO/api/webhooks/goatpay. Esse endereço é um exemplo, a ser substituído pelo domínio real. A função deve validar a assinatura HMAC sobre os bytes originais da requisição e processar cada evento uma única vez.

Os eventos dependem do fluxo escolhido: payment.paid para cobrança Pix; payment_link.paid para checkout de link. Trate também falhas, expiração e reembolso, conforme o produto integrado. A confirmação deve atualizar o pedido no banco. Voltar do checkout não comprova pagamento. [Webhooks oficiais](https://docs.goatpay.com.br/api-reference/guides/webhooks).

Configure o retorno do checkout para pagamento.html?pack=ultra&retorno=1, trocando ultra pelo identificador do pack comprado. O servidor deve preservar a associação com a referência do pedido.

Só quando as rotas estiverem funcionando, altere commerce.apiBase no public/config.js:

```js
apiBase: '/api',
```

Mantenha methods: ['pix']. Confirme que checkoutOrigins contém a origem HTTPS real do checkout retornado pelo servidor. Atualize o repositório e aguarde o deploy.

## 8. Testar a compra e a entrega

1. Verifique pedidos pendentes: não devem revelar links de produto.
2. Faça a compra de teste permitida pelo provedor. A documentação GoatPay consultada informa operação em produção, sem sandbox; nesse caso, teste conscientemente com uma transação real de pequeno valor conforme permitido pela conta.
3. Confirme o recebimento do webhook e a atualização do pedido no banco.
4. Veja o foguete e o botão Acessar pack após a aprovação.
5. Confira no chat o tutorial e a pasta do produto comprado.
6. Repita o fluxo para os três packs e confira os preços.
7. Confira rejeição, expiração, reembolso, notificação repetida e token inválido: não devem gerar liberações indevidas ou entregas duplicadas.
8. Verifique que o e-mail de acesso chega, se o envio tiver sido configurado.
9. Confira que arquivos privados não podem ser abertos diretamente pelo domínio da loja.

As pastas do Drive e os vídeos do YouTube fornecidos são links comuns. Revise suas permissões. Eles podem ser compartilhados por quem recebe o link; ocultá-los até a aprovação não cria proteção individual ou revogação no Drive/YouTube.

Depois disso, mudanças enviadas ao GitHub passam pelo deploy conectado da Vercel. O pacote atual permite realizar as etapas de publicação; a etapa de pagamento só estará completa quando o servidor, o banco e o webhook forem implementados e testados.

# Guia de Deploy em Produção — Craft App Suite Portal

O pacote limpo e pronto para produção já foi gerado na pasta **`dist-deploy/`**:
- **Arquivos reduzidos**: de `634` para **`188` arquivos essenciais**.
- **Tamanho reduzido**: de `~806 MB` para **`413,48 MB`** (ignorando `COMPACTTS/`, scripts locais e 168 MB de cache residual do compilador Rust em `lightcraft-web-0.6.0`).

---

## ⚠️ Nota Importante sobre Limite de 25 MB no Cloudflare Pages
Sete dos módulos `.wasm` (como `effectcraft_web_bg.wasm` com 57,2 MB e `vectorcraft` com 53,7 MB) ultrapassam o limite estrito de **25 MB por arquivo individual** do Cloudflare Pages gratuito/Pro.

Por isso, as duas melhores arquiteturas na **Cloudflare** ou **AWS** são:
1. **Cloudflare R2 + Custom Domain (ou CDN)** — *Sem limite de 25 MB e sem custo de tráfego (Zero Egress).*
2. **AWS S3 + Amazon CloudFront** — *Sem limite de tamanho e com compressão Brotli/Gzip nativa na borda.*

---

## ☁️ Opção 1: Deploy no Cloudflare R2 (Recomendado na Cloudflare)

1. No painel da Cloudflare, acesse **R2 Object Storage** → **Create Bucket** (ex.: `craft-suite-web`).
2. Na aba **Settings** do Bucket:
   - Em **Public Access / Custom Domains**, conecte seu domínio ou subdomínio (ex.: `studio.seudominio.com`).
   - *(Opcional)* Para que acessar `https://studio.seudominio.com/` carregue `index.html` automaticamente na raiz e nas subpastas, ative uma **Transform Rule (Rewrite URL)** no Cloudflare apontando caminhos terminados em `/` para `/index.html`.
3. Para enviar os arquivos com o MIME Type correto (`application/wasm`), configure suas credenciais S3 do R2 no AWS CLI e execute:

```powershell
.\deploy-cloud.ps1 -Action r2 -Bucket "craft-suite-web" -R2AccountId "SEU_ACCOUNT_ID_CLOUDFLARE"
```

---

## 🟠 Opção 2: Deploy no AWS S3 + CloudFront

1. Crie um bucket no **Amazon S3** (ex.: `craft-suite-web`).
2. Crie uma distribuição **Amazon CloudFront** apontando para o bucket S3:
   - **Default root object**: `index.html`
   - **Compress objects automatically**: `Yes` (ativa Gzip e Brotli automaticamente para `.wasm` e `.js`, reduzindo o download de ~57 MB para ~12 MB).
3. Execute o script automatizado para fazer o upload a partir da pasta `dist-deploy` (ele aplica `Content-Type: application/wasm` nos `.wasm` e `Cache-Control: no-cache` nos `index.html`):

```powershell
.\deploy-cloud.ps1 -Action s3 -Bucket "craft-suite-web" -CloudFrontDistId "SEU_DISTRIBUTION_ID"
```

---

## 🔄 Como regerar o pacote `dist-deploy` após alterações no Portal

Sempre que você editar o `index.html` da raiz, basta rodar:

```powershell
.\deploy-cloud.ps1 -Action stage
```
Isso atualizará a pasta `dist-deploy/` limpa e pronta para upload.

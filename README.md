# Política de Privacidade - Clipei

Espelho estático da política de privacidade do app Clipei, hospedado no GitHub Pages.

Motivo: a hospedagem original (HostGator) tem uma regra de ModSecurity que devolve
`406 Not Acceptable` para requisições com User-Agent genérico, o que pode estar
impedindo a validação da URL pelo Google Play. GitHub Pages não tem WAF, então
elimina esse risco.

## Como publicar no GitHub Pages

1. Crie um repositório no GitHub (público, pode ser `politica-de-privacidade` ou
   qualquer nome) e faça push deste conteúdo para a branch `main`.
2. No repositório, vá em **Settings → Pages**.
3. Em "Build and deployment" → Source, selecione **Deploy from a branch**.
4. Branch: `main`, pasta: `/ (root)`. Salvar.
5. Em alguns minutos a página estará disponível em:
   `https://<seu-usuario-github>.github.io/politica-de-privacidade/`

## Usando um domínio próprio (opcional, recomendado)

Para manter a marca (ex: `politica.clipei.pro`) em vez do domínio `github.io`:

1. No provedor de DNS do domínio `clipei.pro`, crie um registro **CNAME**:
   - Nome: `politica`
   - Valor: `<seu-usuario-github>.github.io`
2. Na raiz deste repositório, crie um arquivo `CNAME` (sem extensão) contendo
   apenas: `politica.clipei.pro`
3. Em Settings → Pages, no campo "Custom domain", informe `politica.clipei.pro`
   e aguarde a verificação de DNS. Marque "Enforce HTTPS" quando disponível.
4. Atualize a URL da Política de Privacidade no Play Console para
   `https://politica.clipei.pro/` e reenvie para revisão.

## Atualizando o conteúdo

O arquivo `index.html` é uma cópia exata do que está publicado em
`clipei.pro/politica-de-privacidade`. Sempre que a política mudar no site
principal, replique a alteração aqui também (ou considere manter apenas
esta versão como fonte única, apontando o link em ambos os lugares para ela).

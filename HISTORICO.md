# Histórico: Política de Privacidade rejeitada pelo Google Play

Este documento existe para não precisar reconstruir todo o contexto do zero numa
próxima conversa. Leia isto antes de investigar de novo um problema parecido.

## Linha do tempo

- **2026-09-23** — Google Play reprovou o app Clipei (`pro.clipei.app`) com o erro:
  > "URL provided https://clipei.pro/politica-de-privacidade does not link to a
  > valid privacy policy page"
  >
  > Prazo original: 23/10/2026.

- **Investigação** — a página em si estava correta e completa (política LGPD
  completa, contato, etc). O problema era **infraestrutura**: o servidor da
  **HostGator** (onde `clipei.pro` está hospedado) tem uma regra de
  **ModSecurity** que devolve **HTTP 406 "Not Acceptable"** para requisições
  cujo User-Agent seja genérico/curto (ex: `Mozilla/5.0` sozinho, sem o resto
  da string que navegadores reais enviam).

  Teste que comprova (rodar de novo se suspeitar do mesmo problema em qualquer
  URL hospedada na HostGator):
  ```bash
  curl -s -o /dev/null -w "%{http_code}\n" -A "Mozilla/5.0" https://SEU_DOMINIO/caminho
  # 406 = bloqueado pelo ModSecurity da HostGator
  # 200 = ok
  ```
  Curiosamente, os User-Agents *reais* do Google (`Storebot-Google`,
  `Googlebot`, `APIs-Google`) sempre passaram com 200, então esse bloqueio
  específico **provavelmente nunca foi a causa raiz** da reprovação — mas é um
  problema de segurança real do host que vale continuar cobrando da HostGator.

- Abrimos chamado na HostGator pedindo ajuste/whitelist do ModSecurity. O
  suporte "resolveu" o chamado, mas o reteste (mesmo dia seguinte) mostrou
  **o mesmo bloqueio 406, sem nenhuma mudança**.

- **Decisão:** em vez de continuar dependendo da HostGator, migramos a
  política de privacidade para um espelho estático no **GitHub Pages**
  (sem WAF, sem PHP, 100% estático) — elimina essa categoria de problema.

## Onde está tudo hoje

- **Repositório:** https://github.com/VitinIMORRIVEL/politica-de-privacidade
  (conta GitHub do usuário: `VitinIMORRIVEL`)
- **Clone local:** `C:\Users\vgcon\OneDrive\Documentos\Clipei\politica-de-privacidade`
- **Conteúdo original (fonte histórica, site antigo na HostGator):**
  `C:\Users\vgcon\OneDrive\Documentos\Clipei\site\politica-de-privacidade.html`
  (o `index.html` deste repo é uma cópia dele — se a política mudar, atualize
  os dois lugares, ou considere manter só este repo como fonte única).
- **GitHub Pages:** ativado via API (`gh api repos/.../pages`), branch `main`,
  pasta raiz.
- **Domínio customizado:** `politica.clipei.pro` (arquivo `CNAME` neste repo).
  - DNS gerenciado no painel da **HostGator** (nameservers
    `nspro00162.hostgator.com.br`), mesmo domínio `clipei.pro` do site antigo.
  - Registro necessário: **CNAME** `politica.clipei.pro` → `vitinimorrivel.github.io`
  - **Pegadinha que já aconteceu:** já existia um registro **A** antigo para
    `politica.clipei.pro` (apontando pro IP do site antigo, `69.6.213.99`),
    provavelmente criado antes via cPanel > Subdomínios. CNAME não pode
    coexistir com outro tipo de registro no mesmo nome — foi preciso
    **excluir o A antes de criar o CNAME**. Se o domínio quebrar de novo,
    confira se algum registro conflitante não voltou a aparecer.
  - Certificado HTTPS: emitido automaticamente pelo GitHub (Let's Encrypt) após
    a propagação do DNS. Pode demorar de minutos a ~1h por causa de cache de
    DNS antigo (TTL) em resolvers pelo mundo. Se demorar demais, forçar
    reverificação via API ajuda:
    ```bash
    gh api repos/VitinIMORRIVEL/politica-de-privacidade/pages -X PUT -f "cname="
    gh api repos/VitinIMORRIVEL/politica-de-privacidade/pages -X PUT -f "cname=politica.clipei.pro"
    ```
  - Checar status do certificado/enforcement:
    ```bash
    gh api repos/VitinIMORRIVEL/politica-de-privacidade/pages
    # olhar: https_certificate.state (deve ser "approved") e https_enforced (true)
    ```

- **URL atual cadastrada no Play Console:** `https://politica.clipei.pro/`

## Segunda rejeição (2026-09-26)

Mesmo depois da migração, chegou um novo e-mail do Google Play com o **mesmo
erro genérico**, agora citando a nova URL:
> "URL provided https://politica.clipei.pro/ does not link to a valid privacy
> policy page" — novo prazo: 26/10/2026.

Retestamos tudo (200 OK com e sem barra final, sem `robots.txt` bloqueando,
sem meta `noindex`, sem `X-Frame-Options`/CSP problemático, certificado válido,
`Storebot-Google` retornando 200). **Nenhum problema técnico encontrado.**

**Hipótese mais provável:** o envio para revisão em 24/09 foi feito bem na
janela em que DNS/certificado ainda estavam se estabilizando (tínhamos acabado
de forçar a reemissão do certificado minutos antes). O crawler do Google
provavelmente bateu na URL exatamente nessa janela de instabilidade e marcou
como inválida antes de tudo propagar 100%.

**Ação tomada:** reenviar para revisão sem mudar a URL (ela já está correta),
já que agora está tudo estável há dias. Se rejeitar de novo com o mesmo motivo
mesmo com tudo tecnicamente saudável, o próximo passo é abrir chamado
diretamente com o suporte do Google Play (Play Console → Ajuda), pois nesse
ponto o problema não seria mais de infraestrutura nossa.

## Checklist rápido para a próxima vez

1. `curl -sI https://politica.clipei.pro/` → esperar `200 OK`.
2. `curl -A "Mozilla/5.0" -o /dev/null -w "%{http_code}" https://politica.clipei.pro/`
   → esperar `200` (se der 406, o problema voltou a ser um WAF/ModSecurity,
   improvável no GitHub Pages, mas confira se não migrou o DNS de volta pra
   HostGator por engano).
3. `gh api repos/VitinIMORRIVEL/politica-de-privacidade/pages` → confirmar
   `status: "built"`, `https_certificate.state: "approved"`, `https_enforced: true`.
4. Se tudo isso estiver OK e o Google ainda reprovar, é hora de contatar o
   suporte do Google Play diretamente — não é mais um problema de hospedagem.

## Autenticação usada

- `gh` (GitHub CLI) instalado em `C:\Program Files\GitHub CLI\gh.exe`,
  autenticado via `gh auth login` (browser flow) como `VitinIMORRIVEL`.
  Se abrir um terminal novo e `gh` não for reconhecido, é só o PATH da sessão
  atual não ter sido atualizado — abrir um terminal novo resolve, ou chamar
  pelo caminho completo do executável.

# Tutorial: Deploy no BeamUp do zero até o push

Guia reutilizável para publicar qualquer projeto Node.js no BeamUp (serviço gratuito da Stremio, baseado em Dokku/Herokuish).

---

## 1. Pré-requisitos

- Node.js instalado na máquina
- Conta no GitHub
- Sua chave SSH cadastrada na conta do GitHub (é assim que o BeamUp autentica você)

## 2. Instalar o CLI do BeamUp

```bash
npm install beamup-cli -g
```

## 3. Preparar o projeto para ser reconhecido automaticamente

O BeamUp usa o **buildpack automático do Node** (via Herokuish) — ele detecta seu app sozinho, **desde que**:

- exista um `package.json` na raiz do projeto
- o campo `scripts.start` aponte para o arquivo de entrada, ex:
  ```json
  {
    "name": "meu-projeto",
    "main": "app.js",
    "scripts": {
      "start": "node app.js"
    }
  }
  ```
- o servidor leia a porta HTTP da variável de ambiente `PORT` (não fixe uma porta hardcoded)

> ⚠️ **Não inclua um `Dockerfile` no repositório**, a menos que o nome do seu projeto no BeamUp contenha a palavra "docker". Se houver um `Dockerfile`, o Dokku tenta buildar por ele em vez de usar o buildpack automático, e isso costuma quebrar o processo de start (erro clássico: `Cannot find module '/start'`). Se seu projeto é Node puro, é mais simples deletar o Dockerfile e deixar o buildpack automático cuidar de tudo.

## 4. Iniciar o repositório Git (se ainda não existir)

```bash
git init
git add .
git commit -m "primeiro commit"
```

## 5. Configurar o CLI (uma vez por máquina)

```bash
beamup config
```

Ele vai pedir:
- **host**: use `a.baby-beamup.club`
- **usuário do GitHub**

Isso fica salvo e não pergunta de novo, a menos que você rode `beamup config` outra vez.

## 6. Rodar o setup + primeiro deploy

Dentro da pasta do projeto:

```bash
beamup
```

O comando `beamup` é universal — ele detecta se é a primeira vez (cria o app no servidor e adiciona o remoto Git local chamado `beamup`) ou se já existe (e só faz o deploy). Acompanhe a saída linha por linha: se algo falhar no meio (SSH, buildpack, etc.), o processo pode abortar antes de criar o remoto.

Ao final, ele imprime a URL pública do app, no formato:
```
<hash>-<nome-do-projeto>.baby-beamup.club
```

## 7. Confirmar que o remoto foi criado

```bash
git remote -v
```

Deve aparecer algo como:
```
beamup  dokku@a.baby-beamup.club:<hash>/<nome-do-projeto> (fetch)
beamup  dokku@a.baby-beamup.club:<hash>/<nome-do-projeto> (push)
```

## 8. Deploys seguintes (depois de alterar código)

```bash
git add .
git commit -m "sua mensagem"
git push beamup master
```

> Se sua branch local não for `master` (ex: `main`), use:
> ```bash
> git push beamup main:master
> ```
> O lado direito precisa ser sempre `master`, porque é isso que o Dokku espera receber.

## 9. Verificar se o app está no ar

Abra no navegador (ou via `curl`):
```
https://<hash>-<nome-do-projeto>.baby-beamup.club
```
Para addons Stremio, teste o manifesto:
```
https://<hash>-<nome-do-projeto>.baby-beamup.club/manifest.json
```

## 10. Comandos úteis do dia a dia

```bash
beamup logs              # ver logs do app
beamup secrets <nome> <valor>   # adicionar variável de ambiente/secret
beamup delete             # apagar o projeto completamente
beamup config             # reconfigurar host/usuário GitHub
```

---

## Checklist rápido para um projeto novo

1. `npm install beamup-cli -g` (só na primeira vez na máquina)
2. Confirmar `package.json` com `scripts.start` certo
3. **Sem Dockerfile** no repo (a menos que o nome tenha "docker")
4. `git init && git add . && git commit -m "..."`
5. `beamup config` (se ainda não configurado nesta máquina)
6. `beamup` → cria o app e faz o primeiro deploy
7. `git remote -v` → confirma o remoto `beamup`
8. Dali em diante: `git add . && git commit -m "..." && git push beamup master`

## Erros comuns e como resolver

| Erro | Causa provável | Solução |
|---|---|---|
| `fatal: 'beamup' does not appear to be a git repository` | O remoto `beamup` não existe nesta cópia local | Rode `beamup` de novo, ou adicione manualmente: `git remote add beamup dokku@a.baby-beamup.club:<projeto>` |
| `Cannot find module '/start'` | Existe um `Dockerfile` no repo, conflitando com o buildpack automático | Remova o `Dockerfile` (a menos que o nome do projeto tenha "docker") |
| `Everything up-to-date` (sem novo deploy) | Não há nenhum commit novo em relação ao que já está no servidor | Faça um commit com as mudanças antes do `git push beamup master` |

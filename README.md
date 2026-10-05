# Sasfinanças

Aplicação React + Vite + TypeScript + Tailwind CSS, em Português Europeu. Tema escuro, gestão exclusivamente pessoal, resumo financeiro, gráficos com dados reais, movimentos pontuais/mensais/anuais, objetivos, planeamento e PWA.

## Executar

Node 22.13 ou superior; pnpm 11.

```sh
pnpm install
cp .env.example .env
pnpm dev
```

Sem credenciais, a interface abre num estado neutro, sem dados de exemplo. Os controlos de gravação ficam desativados.

## Configurar Supabase

1. Cria um projeto Supabase.
2. Executa `supabase/schema.sql` no editor SQL. Não contém dados iniciais. As políticas RLS restringem os registos à conta autenticada.
3. Ativa autenticação por email/palavra-passe e configura o URL da aplicação e os URLs de redirecionamento para confirmação de email.
4. Preenche `.env` com `VITE_SUPABASE_URL` e `VITE_SUPABASE_ANON_KEY`. Usa apenas a chave pública anon. Nunca uses a chave service_role no frontend.
5. Reinicia o servidor ou volta a compilar. As variáveis Vite são incorporadas na compilação.
6. Abre a Área Pessoal, cria uma conta com nome/email/palavra-passe, confirma o email se solicitado e inicia sessão.
7. A Área Pessoal permite alterar nome, email e palavra-passe. Para alterar a palavra-passe, confirma a atual. Alterações de email podem exigir confirmação nos dois endereços.
8. Executa também o bloco de publicação Realtime do SQL se já tinhas criado as tabelas: habilita alterações automáticas. A aplicação subscreve inserções/atualizações por user_id; eliminações originam nova leitura protegida por RLS. Existem verificações de segurança de 30 em 30 segundos e ao regressar à janela.

A sessão não é persistida no dispositivo. Recarregar a aplicação exige novo início de sessão. O SDK usa uma sessão em memória; os dados financeiros são gravados exclusivamente no Supabase. Não existe armazenamento financeiro no navegador. O service worker guarda apenas a interface e os recursos estáticos, nunca pedidos ao Supabase.

## Regras financeiras

- Saldo POS e reserva são valores atuais introduzidos pelo utilizador. Não são alterados automaticamente pelos movimentos, evitando duplicar valores já incluídos no saldo.
- Meta Real do resumo = saldo POS + salvaguarda. Os cartões abrem a edição de saldos ou a gestão de entradas/despesas diretamente no painel. Os saldos e a Meta Real apresentam uma pré-visualização durante a edição; a gravação requer Supabase e sessão.
- Entradas/despesas mensais são calculadas a partir dos movimentos. Recorrências mensais contam a partir do mês inicial; anuais contam no mês de vencimento, a partir do ano inicial.
- A provisão mensal de subscrições anuais é o total anual dividido por 12; não é uma segunda despesa.
- Meta de cada objetivo = produto + salvaguarda. Progresso = capital atribuído / meta real, limitado visualmente a 100%.
- Capital dos objetivos é uma atribuição informativa e não é somado novamente ao capital acumulado.
- Os gráficos representam o orçamento por mês, incluindo recorrências; não simulam um extrato bancário nem confirmação de pagamento.
- Para terminar uma recorrência, edita ou elimina o registo. A aplicação não faz débitos ou pagamentos.

## Google Calendar

Cada compromisso abre a página de criação de evento com título, data e valor. O utilizador confirma no Google Calendar. Não existe OAuth, acesso ao calendário, sincronização bidirecional ou criação automática.

## PWA

Manifesto, ícones de 192/512 px e service worker incluídos. `Instalar App` abre o pedido nativo quando disponível; caso contrário, apresenta instruções específicas do navegador. Requer HTTPS e suporte de instalação. Em modo offline, a interface pode abrir, mas os dados e a gravação exigem ligação. Uma instalação não contorna o início de sessão.

## GitHub / Vercel

O código é um projeto Vite autónomo. Cria um repositório GitHub e envia o conteúdo do projeto, excluindo `.env`, `node_modules`, `dist` e `.sites-runtime` (já ignorados).

Importa o repositório no Vercel: framework Vite; comando `pnpm build`; saída `dist`. Adiciona as duas variáveis públicas nas definições do projeto e publica. `vercel.json` está incluído. Para GitHub Pages, adapta `base` no Vite caso uses um subdiretório e configura o endereço inicial do manifesto.

```sh
pnpm check
pnpm build
pnpm preview
```

## Verificação

A compilação verifica tipos TypeScript e gera a PWA. Sem um projeto Supabase configurado não é possível validar operações de gravação, políticas RLS ou autenticação num serviço real. Após ligar: cria duas contas e verifica o isolamento; testa criar/editar/eliminar movimentos e objetivos; testa desconexão de rede e falha de gravação (o formulário deve permanecer preenchido); confirma instalação num navegador compatível.

## Sessão e gravação

A Área Pessoal mostra o nome guardado nos metadados do Supabase, o email e o estado da sessão. O cabeçalho é clicável: sem sessão, apresenta o convite para entrar; com dados carregados e gravados, mostra o email e o estado de nuvem. Durante gravações ou falhas, não afirma que está sincronizado. Renovar o token ou alterar o perfil não apaga os dados carregados. Terminar sessão ou trocar de conta limpa o ecrã e os rascunhos financeiros.

Saldos são gravados por UPSERT com a chave composta user_id/scope; alterações de movimentos usam UPDATE filtrado por id/user_id; novos registos usam INSERT com user_id. As políticas RLS continuam a validar a propriedade na base de dados. Cada gravação é confirmada antes de apresentar sucesso; o formulário e os valores introduzidos permanecem se a escrita falhar. As palavras-passe são enviadas apenas ao Supabase Auth e não são guardadas nas tabelas financeiras.

## Testar ligação ao Supabase

Nas Definições, o botão **Testar Ligação ao Supabase** executa quatro testes sequenciais: validação do URL/chave pública (aceita sb_publishable e JWT anon), HTTP ao endpoint Auth health, SELECT nas três tabelas pessoais e UPSERT numa tabela de diagnóstico. O teste de escrita verifica criação e atualização, lê o registo e confirma a eliminação. As falhas apresentam o motivo e nunca são convertidas em sucesso. Não altera valores financeiros.

Executa o bloco `sf_connection_tests` de `supabase/schema.sql` antes de testar a escrita. A tabela tem políticas RLS e registos associados a user_id. Se a rede cair durante a limpeza, o painel mostra o identificador do registo que pode ter ficado retido; este registo não aparece nos movimentos. Leitura/escrita exigem sessão na Área Pessoal. Nenhuma sessão é criada automaticamente pelo diagnóstico.

As credenciais são lidas exclusivamente de `import.meta.env.VITE_SUPABASE_URL` e `import.meta.env.VITE_SUPABASE_ANON_KEY`. O ficheiro .env não está incluído no código partilhado. Ao publicar noutro fornecedor, configura ambas as variáveis e recompila. Variáveis de execução de Sites são preservadas no serviço; uma aplicação Vite estática precisa das variáveis também durante a compilação.

## Login Google

A Área Pessoal inclui Continuar com Google, via `signInWithOAuth({provider:'google'})`. A aplicação usa o fluxo implicit, deteta a sessão no URL de retorno e mantém a sessão apenas em memória. Não persiste tokens no dispositivo. Após o retorno, o SDK remove o fragmento de autenticação e a aplicação remove o parâmetro de callback. Erros de autenticação são apresentados na Área Pessoal.

Ativa Google no Supabase com um Client ID / Client Secret Google OAuth para uma aplicação Web. Configura a origem da app no Google Cloud, a URI de callback indicada pelo Supabase e o Site URL / Redirect URLs da app no Supabase. Adiciona também o endereço `/?auth=callback`. O Client Secret Google pertence apenas às definições do fornecedor Supabase e nunca às variáveis Vite.

A verificação de fornecedores só lê `/auth/v1/settings`. Caso Google esteja desativado, o botão apresenta instruções em vez de afirmar que iniciou sessão. Email e palavra-passe continuam disponíveis. Utilizadores que iniciam sessão exclusivamente com Google podem definir uma palavra-passe nas credenciais do perfil. Os testes de leitura/escrita continuam a exigir respostas reais da base de dados. Login bem-sucedido não converte erros de tabela, RLS ou rede em sucesso.

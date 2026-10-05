# Publicação do Sasfinanças

Código completo exportado do projeto validado. Não inclui .env, credenciais, node_modules ou dados financeiros.

1. Extrair o ZIP e colocar os ficheiros na raiz de um repositório GitHub privado, incluindo .env.example, .gitignore e pnpm-lock.yaml.
2. Importar esse repositório no Netlify ou Vercel.
3. Node 22; gestor pnpm (versão declarada em package.json); build pnpm build; saída dist.
4. Definir VITE_SUPABASE_URL e VITE_SUPABASE_ANON_KEY no ambiente de compilação. Esta última aceita a chave pública sb_publishable. Nunca usar service_role ou sb_secret.
5. Publicar/recompilar após definir ou alterar variáveis.
6. Supabase Authentication > URL Configuration: Site URL = domínio final HTTPS; Redirect URLs = domínio final seguido de / e /?auth=callback.
7. Para Google, adicionar a nova origem no Google Cloud; manter callback do projeto Supabase. Ativar fornecedor Google no Supabase se necessário.
8. Iniciar sessão no domínio final; executar quatro testes; criar/editar/apagar um movimento real e verificar após novo login; testar instalação no telemóvel.

netlify.toml e vercel.json incluem revalidação dos ficheiros de entrada/PWA. O build gera assets com hash e uma versão de cache derivada dos assets. Não existe garantia absoluta contra problemas de cache em todos os navegadores.

A sessão fica apenas em memória: recarregar exige novo login. A interface offline não permite ler/gravar finanças sem rede. A instalação depende do navegador; em iPhone usar Adicionar ao ecrã principal.

Para upload manual Netlify, compilar localmente com as variáveis e enviar a pasta dist completa. Variáveis configuradas no painel não alteram ficheiros já compilados.

Os dados e utilizadores continuam no mesmo Supabase. GitHub versiona código, não faz backup da base de dados. A validação de isolamento entre contas e de Realtime deve ser feita separadamente dos quatro testes.

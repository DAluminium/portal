# Integração local da segurança D’aluminium

Worker completo e cinco páginas adaptadas preparados localmente. **Nada foi publicado. Não cole o Worker isoladamente em produção.** O pacote anterior de fundação foi ampliado; este documento descreve o estado atual.

23 testes locais aprovados com banco, limites de login e funções de senha simulados. Sintaxe dos scripts dos cinco clientes conferida. Ainda falta validar SQL/bcrypt no Supabase, runtime Cloudflare e fluxos completos no navegador em infraestrutura isolada.

## Arquivos

Worker.integrado.js é a versão única para avaliação no editor Cloudflare. worker.mjs e security.mjs são as fontes organizadas. A pasta clientes contém os index.html de Portal, Compras, Financeiro, Fábrica e Obra com o adaptador embutido. migration-passwords.sql é o pré-requisito SQL de senhas para revisão e aplicação primeiro em teste. Arquivos test.mjs reproduzem os testes locais.

## Regras integradas

Admin tem acesso total. Apenas admin concede, retira ou muda módulos. Admin e gerente cadastram usuários; gerente só cria cargos abaixo do seu nível e usuários novos sem módulos. Obra só permite gravação por admin ou gerente com Obra autorizado. Consulta só lê dados operacionais, podendo gerenciar seus próprios lembretes e presença. Os demais cargos gravam nos módulos concedidos. Ambientes de teste continuam limitados a admin e gerente.

## Proteções

Sistemas desconhecidos recusados; consultas estruturadas; CORS exato; token assinado com uma hora de validade; cadastro atual consultado para autorização e revogação após mudanças de senha ou acesso; sem fallback de senha administrativa em variáveis Cloudflare.

Cadastro separado em users-list e users-save. Senhas armazenadas não são devolvidas aos clientes; a lista protegida é preservada em gravações operacionais. Gerente não altera módulos nem cria admin. Portal só confirma alteração de senha depois de persistência confirmada.

Sessões e lembretes de não administradores são filtrados por proprietário e mesclados no servidor. Gravação exige versão do documento lido e PATCH condicional atômico no banco, retornando 409 para conflito e 428 sem versão. Polling posterior não autoriza payload antigo.

Senhas legadas são migradas no login válido para bcrypt custo 12 sobre o digest SHA-256 compatível com o frontend. As RPCs usam SECURITY INVOKER e search_path vazio, com execução revogada de PUBLIC, anon e authenticated e concedida apenas a service_role. Digests enviados são credenciais: usar HTTPS e não registrar corpos em logs. Geração e verificação reais no banco ainda não testadas.

## Pré-requisitos de teste real

1. Ambiente isolado com dados fictícios. Os arquivos mantêm os endereços atuais: antes de executar, substituir BASE do Worker, endpoints do Worker e URLs dos módulos nos clientes, endpoint do adaptador e origem CORS pelos valores de teste. Não usar service_role de produção em teste.
2. Verificar pgcrypto no schema extensions e aplicar as RPCs em teste. Confirmar recusa de execução por anon/authenticated e sucesso por service_role.
3. Conferir app_data, IDs, updated_at, triggers e suporte ao PATCH condicional. Não executar restaurações.
4. Configurar SUPA_SECRET de teste, TOKEN_SECRET aleatório de pelo menos 32 bytes e bindings LOGIN_IP_LIMIT e LOGIN_USER_LIMIT. Sugestão inicial: 10/minuto por IP e 5/minuto por usuário; ajustar com uso legítimo. Limites Cloudflare são aproximados por localização, não contador global rigoroso.
5. Validar login, senha temporária, troca de senha, cadastro, remoção, desativação, módulos, integração Obra → Compras/Fábrica, sessões, lembretes e conflitos. Gerente precisa também ter os módulos de destino autorizados para integrações; não há exceção implícita.
6. Telas legadas de cadastro/troca de senha em Compras e Financeiro precisam ser redirecionadas para o portal; gravações de usuários por esses módulos serão recusadas. Fluxos que reconstruam objetos sem preservar _securityVersion precisam de ajuste específico antes de promoção.
7. Conferir commits publicados, backup restaurável e rollback. Conector GitHub está somente para leitura; sessão do navegador autenticada como proprietária. Ramo security/restruturacao-20260914 criado para revisão, sem promoção para main.

## Restante da reestruturação

Validação real e publicação; política mais forte de senha e MFA/Supabase Auth; revogação imediata por encerramento administrativo de sessão; validação de esquema por operação; auditoria persistente; revisão completa de innerHTML; permissões visuais de Obra; funções SECURITY DEFINER/EXECUTE/search_path existentes; backups externos e restauração testada; varredura semanal. Não há garantia de segurança máxima.

Referências: https://developers.cloudflare.com/workers/runtime-apis/bindings/rate-limit/ e https://supabase.com/docs/guides/auth/password-security


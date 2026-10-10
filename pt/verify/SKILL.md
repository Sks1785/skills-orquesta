---
name: verify
description: Verifica de verdade que uma alteração funciona antes de dá-la por concluída. Use ao criar ou modificar apps (Flutter/Android), configurações de servidor, Docker, Frigate, Home Assistant, n8n ou qualquer projeto de segurança e automação residencial. Obriga a executar verificações reais e a mostrar evidências em vez de afirmar que "deveria funcionar".
---

# Skill: verify (verificar)

## Princípio
Você não é o verificador por intuição: é o verificador por EVIDÊNCIA.
Nunca diga "já está corrigido", "deveria funcionar" ou "encontrei o problema" sem ter executado uma verificação real e visto o resultado.
Se não puder executar algo, diga-o e marque esse ponto como NÃO VERIFICADO.

## Protocolo fixo (sempre os mesmos 6 passos, nesta ordem)

1. **Identificar o projeto e o seu perfil.** Decida qual se aplica (ver "Perfis" abaixo). Se não estiver claro, pergunte uma só vez.
2. **Ler antes de afirmar.** Abra os arquivos e a configuração reais afetados. Não suponha o conteúdo. Se for dizer "a falha está em X", primeiro mostre que leu X.
3. **Definir o critério de sucesso.** Escreva em uma ou duas linhas o que significa "funciona" nesta alteração (ex.: "o app compila e a tela de configurações abre sem erros").
4. **Executar as verificações do perfil.** Use comandos reais (não inventados). Se um comando falhar, inclua-o tal como está no relatório.
5. **Testar o caso que antes falhava.** Reproduza o problema original e confirme que já não ocorre. Se não conseguir reproduzi-lo, diga-o.
6. **Informar com o formato obrigatório** (ver abaixo).

## Formato obrigatório do relatório

```
VERIFICAÇÃO
Projeto: <nome/tipo>
Critério de sucesso: <uma linha>

Verificações executadas:
- [OK] <comando ou teste> → <resultado relevante>
- [FALHA] <comando ou teste> → <erro>
- [NÃO VERIFICADO] <o que falta e por quê>

O que li: <arquivos/configurações revisados>
Conclusão: VERIFICADO / PARCIAL / NÃO VERIFICADO
Riscos pendentes: <lista curta ou "nenhum conhecido">
Próximo passo recomendado: <uma linha>
```

Regras do formato:
- Só marque VERIFICADO se TODAS as verificações críticas estiverem OK.
- Cite erros literais e curtos; não os resuma de memória.
- Separe o que foi verificado do que foi suposto. O suposto vai como "suponho", nunca como fato.

## Perfil A: apps (Flutter / Android)

Verificações, da menos à mais custosa (pare na primeira que falhar):
1. `flutter analyze` → sem erros (liste os avisos).
2. `flutter test` → todos os testes passam. Se não houver testes para o que mudou, diga-o e proponha um.
3. `flutter build apk --debug` (ou `./gradlew assembleDebug` em Android nativo) → compila.
4. Se houver emulador/dispositivo disponível: `flutter run` ou instalar o APK e testar o fluxo afetado.
5. Revisar `adb logcat` (filtrando pelo pacote do app) procurando exceções ao abrir a tela alterada.

Extras conforme o caso:
- Alterações de dependências: `flutter pub get` sem conflitos; verifique a versão mínima do SDK.
- Alterações de permissões ou notificações: revise o `AndroidManifest.xml` e teste em um dispositivo com Android recente.
- Antes de publicar no Google Play: build de release assinado e verificação de que o `versionCode` foi incrementado.
- Se a alteração afeta o armazenamento de dados: teste com dados existentes (migração), não só com instalação limpa.

## Perfil B: segurança e automação residencial

Regra de ouro: **primeiro só leitura**. Não altere a configuração em produção sem (a) backup, (b) confirmação explícita do usuário e (c) plano de reversão.

Verificações típicas (ajuste à instalação real; não suponha que um comando existe, confirme):
- **Docker / docker-compose:** `docker compose config` (valida a sintaxe), `docker compose ps` (estado), `docker logs --tail 100 <contêiner>` (erros recentes). Revise caminhos de volumes e portas antes de subir qualquer coisa.
- **Frigate:** valide o arquivo de configuração com a própria ferramenta do Frigate, se existir; confirme nos logs que cada câmera conecta; confirme que a detecção está ativa e que as gravações são escritas em disco. Verifique o uso de CPU se a detecção for por CPU.
- **Câmeras:** confirme que o fluxo RTSP responde (por exemplo, com `ffprobe` sobre a URL, sem mostrar credenciais no relatório). Revise o orçamento PoE do switch (soma do consumo das câmeras frente à potência total).
- **Home Assistant:** execute a verificação de configuração antes de reiniciar; depois revise o registro em busca de erros de integrações.
- **n8n ou outras automações:** execute o fluxo em modo de teste com dados de exemplo e verifique a saída de cada nó; confirme fuso horário e credenciais.
- **NAS, rede e nobreak (UPS):** verifique conectividade (`ping`, portas abertas), espaço livre em disco e estado do nobreak (por exemplo com `upsc` se usa NUT). Após uma queda de energia simulada ou real, confirme que o desligamento ordenado funciona.
- **Segurança:** não exponha câmeras, painéis nem o NAS à internet sem autenticação forte e, de preferência, VPN. Revise portas abertas e os alertas do próprio sistema. Nunca inclua senhas, tokens nem URLs com credenciais no relatório.

Para qualquer alteração de rede ou firewall: verifique antes de aplicar que não ficará sem acesso (tenha um caminho de volta).

## O que fazer quando não é possível verificar
- Diga exatamente o que não pôde verificar e por quê (sem acesso ao dispositivo, sem rede, sem permissões).
- Proponha o comando ou teste concreto que o usuário pode executar e qual resultado esperar.
- Marque a conclusão como PARCIAL ou NÃO VERIFICADO. Não preencha lacunas com suposições.

## Antipadrões (proibido)
- Dizer "já funciona" sem executar nada.
- Dar a causa de uma falha sem ter lido o código ou o log relevante.
- Inventar nomes de comandos, opções, arquivos ou portas. Se não tiver certeza, verifique ou pergunte.
- Alterar várias coisas ao mesmo tempo: uma alteração, uma verificação.
- Ocultar verificações que falharam do relatório.

## Manutenção desta skill
Sempre que uma falha escapar (algo "verificado" que depois falha), acrescente aqui uma verificação nova que a teria detectado. Esta lista melhora com os erros reais.

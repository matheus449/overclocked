# TI Company Simulator — v3 (simulação operacional)

Cooperativo para até 14 jogadores, no navegador. Escritório isométrico (Three.js) + servidor autoritativo (Node.js, Express, Socket.io). O foco é triagem, diagnóstico e reparo metódico, sem corrida contra o relógio.

## Executar

```bash
npm install
npm start          # http://localhost:3000   (Node 18+)
```

## Estrutura

```
server/server.js      # clientes NPC, chat/OS, notificações por sala, simulações de bancada
shared/shared.js      # mapa, física determinística, departamentos, catálogos
public/index.html     # canvas + estrutura da UI
public/css/style.css  # cartões brancos, destaques roxos, estilos das simulações
public/js/ui.js       # DOM: modal de chat/OS, bancadas e simulações 2D, banners
public/js/game.js     # Three.js: cena, avatares, NPCs na fila, rede, predição/interpolação
```

## Fluxo

1. **Cliente NPC** chega (a cada ~18 s, até 5 na fila). Só a Recepção recebe `client_arrived`.
2. **Recepção** abre o Balcão de Atendimento, escolhe um cliente e conversa. O NPC responde por palavras-chave (perguntas rápidas ou texto livre). Depois preenche a **OS**: resumo + departamento (Hardware, Software/Segurança ou Redes) → `os_submit`.
3. O servidor cria o PC ligado à OS, põe-no na **Mesa de Triagem** e emite `notify_task` **apenas para a sala `role:<dept>`** (Socket.io rooms). Só esses técnicos veem o banner «Nova máquina aguardando reparo na bancada de triagem».
4. O técnico recolhe a máquina (só pode pegar as do seu departamento), leva-a à bancada e **inspeciona**. Se a triagem estava errada, a inspeção revela o departamento real e o técnico reencaminha (a Recepção recebe feedback e o grupo perde pontos).
5. **Simulação 2D** (cada passo validado no servidor):
   - Hardware (tela quebrada): retirar 4 parafusos → remover tela → escolher a tela certa no inventário (conforme a etiqueta do notebook) → ligar cabo → testar.
   - Software/Segurança (vírus/phishing): fechar popups, abrir o SecureScan, verificar (4 s), pôr em quarentena só as ameaças reais (falsos positivos penalizam) → concluir limpeza.
   - Redes (sem Wi-Fi): `ipconfig`, ping ao gateway/internet, renovar IP ou reiniciar o roteador, conforme a causa (IP inválido ou WAN em baixo) → testar.
6. O PC reparado vai para a **Mesa de Saída**; a Recepção recolhe-o e entrega no **Balcão do Cliente**. A satisfação (1–5 ★) desce com erros e reencaminhamentos.

## Rede (novidades)

| Evento | Destino | Conteúdo |
|---|---|---|
| `client_arrived` | sala `role:reception` | cliente, fila |
| `notify_task` | sala `role:<dept>` | OS, cliente, texto, pendentes |
| `triage_feedback`, `delivered` | sala `role:reception` | reencaminhamento / satisfação |
| `open_station` / `close_station` | jogador | vista montada no servidor (2 Hz enquanto aberta) |
| C→S `chat_open`, `chat_say`, `chat_release`, `os_submit` | — | atendimento |
| C→S `inspect`, `reroute`, `sim {act,arg}`, `bench_collect`, `deliver` | — | bancadas |

O servidor nunca envia segredos ao cliente (qual ficheiro é malicioso, causa da falha de rede); só valida as ações.

## Extensão

Para um novo problema: entrada em `PROBLEMS` (falas e pistas), função em `SIMS`, vista em `simView` e um renderer em `ui.js`.

## Limitações

- Os NPCs só aparecem na fila do balcão (não vão ao balcão de entrega).
- Voz continua protótipo; sem modelos GLTF.
- O texto livre do cliente usa palavras-chave, não um modelo de linguagem.

# Reflexão — Roteiro 03

1. **Stubs e skeletons**

Os *stubs* (cliente) e *skeletons* (servidor) encapsulam respectivamente o marshalling/unmarshalling e o dispatch das chamadas remotas. No `t2_stub_manual/stub_manual.py` o stub serializa a chamada em JSON e aplica *framing* (4 bytes de tamanho) antes de enviar; o skeleton lê o framing, desserializa, despacha para a função e serializa a resposta. Sem esses componentes teríamos que expor manualmente sockets, framing e dispatch em todo cliente/servidor, aumentando acoplamento e repetição de código — além de perder tratamento padronizado de erros (ex: converter exceções em mensagens). Conceito técnico: *marshalling, unmarshalling, framing, dispatch table*.

2. **REST não é RPC**

Uma diferença fundamental é a modelagem: RPC é orientado a ações (chamar procedimentos), REST é orientado a recursos (manipular representações via verbos HTTP). Ex.: para "cancelar um pedido" no estilo RPC você chamaria `proxy.cancelarPedido(id)`; em REST você poderia `POST /pedidos/{id}/cancelamento` ou `PUT /pedidos/{id}` com um estado `status=cancelado`. Em `t1_xmlrpc/cliente_xmlrpc.py` vemos chamadas como funções; em `t3_rest/servidor_rest.py` manipulamos recursos `/produtos`. Conceito técnico: *interface uniforme do REST, verbos HTTP vs chamadas procedurais*.

3. **Evolução de contrato**

Protobuf é compatível por design: adicionar um campo `string unidade = N;` em `RespostaCalculo` é tolerado por clientes antigos porque campos desconhecidos são simplesmente ignorados (e os números de campo preservam a interoperabilidade). Em REST sem schema, a adição pode ser feita sem quebrar clientes se for opcional, mas falta verificação estática — problemas só aparecem em runtime. Conceito técnico: *forward/backward compatibility em Protobuf via números de campo; campos opcionais e ignorados pelos desserializadores*.

4. **Escolha de tecnologia**

Para parceiros externos (APIs públicas) eu recomendaria REST: baixa barreira de entrada, JSON legível, hyperlinks e cache HTTP (códigos de status e semântica bem conhecidos). Para comunicação interna entre microsserviços eu recomendaria gRPC: contrato `.proto` e tipagem forte, serialização binária (Protobuf) e HTTP/2 com streaming eficiente — útil em alta taxa de chamadas internas. Baseio a escolha nas dimensões de `t5_comparativo/comparativo.py`: *serialização, contrato, tipagem e performance*.

5. **Conexão com Labs anteriores (transparência excessiva)**

A transparência do RPC (fazer a chamada remota parecer local) pode esconder falhas distribuídas como latência, falhas parciais e retries automáticos — levando o dev a assumir que operações são instantâneas ou atômicas. Por exemplo, ao trocar uma chamada local por `proxy.calcular(...)` (Tarefa 1) sem considerar timeouts/retries, o desenvolvedor pode introduzir bloqueios ou inconsistências. Conceito técnico: *falhas parciais, latência de rede e necessidade de timeouts/retries explícitos*.

---



---
title: "2-Stage Classification e Constrangimento Social via Tipos: Dois Padrões Geniais do Claude Code"
description: "Como a Anthropic resolveu o problema de segurança em tempo real com um classificador de dois estágios e um nome de tipo TypeScript que é, ele mesmo, um checklist de compliance."
date: "2026-04-03"
tags: ["Claude Code", "segurança", "TypeScript", "classificação", "arquitetura"]
author: "UltraThink"
category: "arquitetura"
rating: 9
importance: "muito alto"
readTime: "8 min"
slug: "classificacao-2-estagios-e-constrangimento-social"
draft: false
---

# 2-Stage Classification e Constrangimento Social via Tipos: Dois Padrões Geniais do Claude Code

Existe uma pergunta que todo sistema de IA com capacidade de agir no mundo real precisa responder dezenas de vezes por minuto: **posso fazer isso?**

Editar esse arquivo. Executar esse comando. Deletar esse diretório. Escrever nesse `.bashrc`. Cada ação passa por um julgamento. E esse julgamento tem um custo — em tempo, em tokens, em latência percebida pelo usuário. Se o julgamento for lento demais, a experiência quebra. Se for raso demais, a segurança quebra.

O Claude Code, da Anthropic, resolveu esse dilema com dois padrões de design que, quando você entende a lógica por trás deles, parecem óbvios em retrospecto — mas só porque são bons o suficiente para parecerem assim.

---

## O Guardião de Duas Velocidades

Imagine o controle de segurança de um aeroporto. Há dois caminhos: a fila rápida, onde a maioria das pessoas passa em segundos após uma varredura básica, e a sala de inspeção detalhada, para os casos que o scanner não conseguiu resolver de imediato. A fila rápida não existe porque o aeroporto é descuidado — existe porque a inspeção completa de cada mala de cada passageiro tornaria o sistema inoperante.

O **2-Stage Classifier** do Claude Code funciona exatamente assim.

Quando o agente precisa classificar uma ação — decidir se deve executá-la diretamente, pedir permissão ao usuário, ou bloquear completamente — o primeiro passo é um julgamento expresso:

```
Stage 1: max_tokens=64, stop_sequences=['</block>']
```

Sessenta e quatro tokens. Com uma sequência de parada que força o modelo a encerrar assim que chega a uma conclusão. Isso não é um LLM "pensando" — é um LLM sendo usado como um classificador binário ultra-rápido, sem a sobrecarga de gerar raciocínio elaborado.

Noventa por cento ou mais das ações são resolvidas aqui. O arquivo é claramente seguro de editar, ou claramente perigoso demais para tocar sem aprovação. O classificador decide em milissegundos e o pipeline segue.

Para os casos que ficam no limbo — ações ambíguas, contextos incomuns, combinações de fatores que tornam a decisão não trivial — entra o Stage 2:

```
Stage 2: max_tokens=4096, chain-of-thought completo
```

Quatro mil e noventa e seis tokens. Chain-of-thought. O modelo pensa em voz alta, pesa os fatores, navega pelas nuances. Esse é o processo completo, o raciocínio detalhado que garante que decisões difíceis sejam tomadas com o rigor que merecem.

---

## A Economia do Pensamento

Existe um conceito na economia cognitiva chamado de "atenção seletiva" — o sistema nervoso humano não processa tudo com a mesma profundidade porque isso seria impossível. Você presta atenção plena às coisas que merecem atenção plena. O resto é processado em modo raso, por heurísticas, por padrões reconhecidos.

O 2-Stage Classifier é a implementação desse princípio em software.

O custo de rodar o Stage 2 para cada ação seria proibitivo — não apenas em termos financeiros, mas em latência. Usuários não esperam quatro segundos para o agente decidir se pode abrir um arquivo. Mas rodar apenas o Stage 1 para tudo seria insuficiente: algumas decisões genuinamente precisam de raciocínio profundo.

A solução é elegante porque reconhece uma verdade assimétrica: **a maioria das situações não é ambígua**. Deletar o diretório raiz do sistema é obviamente perigoso. Ler um arquivo de configuração de projeto é obviamente seguro. O espaço de ambiguidade real é pequeno. O Stage 1 cuida do óbvio — que é a maioria — e libera o Stage 2 para o que realmente precisa dele.

O pipeline completo tem mais de doze etapas: `deny → ask → allow → preapproved`, com verificações em cascata. Mas a maior parte das ações nem chega a percorrer esse caminho inteiro. São interceptadas cedo, decididas rápido, e o sistema segue.

---

## Por Que Isso É Genial

O insight central não é técnico — é econômico. É a percepção de que **o custo de uma decisão deve ser proporcional à dificuldade da decisão**.

Sistemas ingênuos aplicam o mesmo nível de escrutínio para tudo. Isso é seguro no sentido de que nenhum caso difícil passa despercebido, mas é ineficiente porque casos fáceis recebem atenção que não precisam. Sistemas rápidos demais sacrificam escrutínio por velocidade e eventualmente deixam algo perigoso passar.

O classificador de dois estágios resolve o trade-off sem eliminá-lo — apenas reposiciona onde o custo é pago. Você paga pouco pela maioria, e paga o preço justo pelos casos que merecem.

E há outro elemento de design que vale destacar: certos arquivos são **bypass-immune**. Arquivos como `.bashrc` e `.gitconfig` nunca podem ser editados sem aprovação explícita do usuário, mesmo quando o agente está rodando em modo `bypassPermissions`. A imunidade ao bypass é uma propriedade hard-coded, não configurável, não contornável por instrução do usuário ou do sistema.

Isso é análogo à "regra de dois homens" em segurança nuclear — certas ações requerem dois agentes independentes autorizando, independentemente de quem está pedindo ou qual o contexto. O sistema recusa-se a confiar em si mesmo para esses casos específicos.

---

## Constrangimento Social via Tipos

Agora, um padrão diferente. Menos sobre arquitetura de sistema e mais sobre engenharia humana.

Existe um tipo TypeScript no código do Claude Code com o seguinte nome:

```typescript
AnalyticsMetadata_I_VERIFIED_THIS_IS_NOT_CODE_OR_FILEPATHS
```

Leia de novo.

`AnalyticsMetadata_I_VERIFIED_THIS_IS_NOT_CODE_OR_FILEPATHS`.

O nome do tipo **é** o checklist de segurança. Não há documentação separada. Não há comentário acima da declaração. Não há reunião de onboarding onde alguém explica "ei, antes de usar esse tipo, certifique-se de verificar que os dados não contêm código ou caminhos de arquivo".

O nome do tipo **é** esse aviso. E, mais do que isso, o nome do tipo é um ato declarativo. Qualquer desenvolvedor que escreve `AnalyticsMetadata_I_VERIFIED_THIS_IS_NOT_CODE_OR_FILEPATHS` em seu código está, literalmente, afirmando no código-fonte que realizou aquela verificação.

---

## Por Que Nomear Assim É Genius

Pense em como sistemas de segurança normalmente funcionam. Há documentação que ninguém lê. Há comentários que ficam desatualizados. Há wikis internas que explicam os invariantes que devem ser respeitados. Há code reviews onde revisores podem ou não pegar o problema. Há reuniões onde o conhecimento institucional é transmitido, mas apenas para quem estava presente.

Todos esses mecanismos têm o mesmo problema: são **separados do ponto de uso**. A documentação existe em outro lugar. O conhecimento existe na cabeça de outra pessoa. O comentário existe algumas linhas acima, ou em outro arquivo, ou em um PR de seis meses atrás.

O tipo `AnalyticsMetadata_I_VERIFIED_THIS_IS_NOT_CODE_OR_FILEPATHS` colapsa essa distância para zero. O invariante a ser respeitado aparece **exatamente no ponto de uso**, toda vez, sem exceção, em cada arquivo onde o tipo é referenciado.

É TypeScript como consciência profissional.

Tem um paralelo claro com o checklist de cirurgiões popularizado pelo livro de Atul Gawande. A ideia por trás do checklist cirúrgico não é que cirurgiões são incompetentes ou esquecidos — é que situações de pressão alta e complexidade alta aumentam a chance de erro mesmo para especialistas. O checklist não substitui o julgamento; ele garante que o julgamento seja aplicado no momento certo.

O nome do tipo faz a mesma coisa no código. Ele não assume que o desenvolvedor vai esquecer de verificar. Ele garante que a verificação seja invocada no momento certo — quando o tipo está sendo usado — sem depender de memória, documentação separada, ou boa vontade.

E há uma dimensão de constrangimento social que é difícil de subestimar. Se um desenvolvedor usa esse tipo e *não* fez a verificação que o nome promete, o código agora contém uma mentira. Uma mentira rastreável, versionada, auditável. Quando aparecer um bug onde dados de analytics continham caminhos de arquivo, alguém vai abrir o git blame, ver quem usou o tipo, e a declaração no nome estará lá: `I_VERIFIED_THIS_IS_NOT_CODE_OR_FILEPATHS`.

O custo social de mentir no nome do tipo é alto. O custo de fazer a verificação é baixo. O design inclina o comportamento na direção certa sem precisar de enforcement externo.

---

## O Que Esses Padrões Ensinam Para o Dia a Dia

Ambos os padrões compartilham uma filosofia subjacente: **mover a segurança para o ponto de menor resistência**.

No 2-Stage Classifier, o ponto de menor resistência é temporal — classificar rápido quando possível, profundo quando necessário. O custo é pago onde precisa ser pago.

No Constrangimento Social via Tipos, o ponto de menor resistência é espacial — o aviso está exatamente onde o código é escrito, não em outro lugar. A atenção do desenvolvedor é capturada no momento certo.

Esses princípios se traduzem diretamente para problemas comuns:

**No design de APIs**: em vez de documentar que certos parâmetros não devem ser usados juntos, use tipos que tornem a combinação inválida impossível de expressar. O compilador substitui a documentação.

**Em pipelines de dados**: em vez de um comentário dizendo "não passe PII para esta função", nomeie o tipo de forma que a função aceite apenas `SanitizedData_PII_REMOVED`. Qualquer dado não sanitizado é um erro de tipo, não um erro humano.

**Em sistemas de autorização**: em vez de verificar permissões em múltiplos lugares, crie um wrapper de tipo que só pode ser instanciado após verificação de permissão. O tipo é a prova de que a verificação aconteceu.

A ideia geral: sempre que você depende de alguém lembrar de fazer algo, procure uma forma de tornar esse algo parte do próprio ato de usar o sistema.

---

## Como Esses Padrões Inspiram a Ter Novas Ideias

O que é mais interessante nesses dois padrões não é o que eles resolvem, mas o que eles revelam sobre como pensar em design.

A maioria dos sistemas de segurança pensa em termos de barreiras: onde colocar um checkpoint? O que bloquear? O 2-Stage Classifier pensa diferente: como fazer o checkpoint custar menos sem comprometer a qualidade? Em vez de reduzir barreiras, reduz o custo de manter barreiras onde elas precisam estar.

O Constrangimento Social via Tipos pensa diferente ainda: em vez de criar barreiras externas ao comportamento, como fazer o comportamento correto ser o caminho natural? Como colocar o prompt de segurança no momento exato em que ele é relevante?

Ambos os padrões são exemplos de **design que constrange o bom comportamento sem proibir o mau comportamento por força**. O classificador não impede que o Stage 2 seja chamado para casos simples — simplesmente não chama, porque o Stage 1 resolveu mais rápido. O nome do tipo não impede que um desenvolvedor ignore o aviso — simplesmente garante que ignorar seja um ato consciente.

Isso é mais robusto do que enforcement puro. Enforcement puro cria incentivo para contornar. Design que torna o bom comportamento conveniente cria incentivo para seguir.

---

## Conclusão

O 2-Stage Classifier e o Constrangimento Social via Tipos são padrões de design com origens completamente diferentes — um é uma otimização de pipeline de ML, o outro é um truque de nomenclatura TypeScript — mas resolvem o mesmo problema fundamental: como garantir que sistemas complexos se comportem corretamente sem tornar o comportamento correto caro ou inconveniente.

A segurança aeronáutica levou décadas para aprender que checklists verbais feitos em voz alta reduzem erros mais do que qualquer tecnologia individual. Cirurgiões levaram mais tempo ainda para aceitar que listas de verificação simples salvam mais vidas do que treinamento adicional. A engenharia de software está aprendendo a mesma lição de formas diferentes.

Um nome de tipo que é um checklist. Um classificador que pensa profundo apenas quando precisa. Dois padrões que, olhando de perto, ensinam mais sobre design de sistemas do que qualquer framework ou biblioteca.

**Rating: 9/10** — Não chegam a 10 porque o verdadeiro teste de um padrão é quanto tempo ele sobrevive e quanto ele se espalha. Esses dois merecem se espalhar.
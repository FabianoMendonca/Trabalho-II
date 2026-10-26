# Trabalho de Sistemas Operacionais: simulação de escalonamento de processos

**Modalidade:** em duplas

## 1. Objetivo

Desenvolver um simulador de escalonamento de processos e comparar o comportamento dos algoritmos estudados em aula. O trabalho deve demonstrar como a escolha do próximo processo afeta o tempo de espera, o tempo de resposta e o turnaround.

A atividade consiste em uma simulação. Não é necessário criar processos reais, utilizar threads ou modificar o escalonador do sistema operacional.

## 2. Algoritmos do trabalho

Serão utilizados somente os algoritmos apresentados em aula:

| Identificador | Algoritmo | Característica |
|---|---|---|
| `FCFS` | First Come, First Served / FIFO | Executa os processos pela ordem de chegada, sem preempção. |
| `SJF` | Shortest Job First | Escolhe o menor tempo de CPU entre os processos prontos, sem preempção. |
| `SRTF` | Shortest Remaining Time First | Escolhe o menor tempo restante, com preempção. |
| `RR` | Round Robin | Alterna os processos prontos utilizando um quantum. |
| `MLFQ` | Filas multinível com realimentação | Utiliza filas de prioridades e altera a posição dos processos conforme sua execução. |
| `CFS` | Completely Fair Scheduler | Busca distribuir a CPU de forma justa utilizando o tempo virtual de execução. |

Cada dupla deverá implementar **FCFS e um segundo algoritmo**, escolhido em acordo com o professor. As opções para o segundo algoritmo são SJF, SRTF, RR, MLFQ e CFS.

Para MLFQ e CFS, deverão ser utilizadas as versões didáticas definidas neste enunciado. A versão de CFS não pretende reproduzir todos os detalhes de uma implementação de kernel.

## 3. Modelo da simulação

Todas as implementações deverão seguir as mesmas condições:

- Uma única CPU.
- Tempo simulado em unidades inteiras, começando em zero.
- Cada processo possui um instante de chegada e um tempo total de CPU necessário para concluir.
- Não existem operações de entrada/saída nem bloqueios durante a execução.
- O tempo total de CPU de cada processo é conhecido na entrada da simulação.
- O custo de troca de contexto é zero.
- A CPU não fica ociosa enquanto houver algum processo pronto.
- Um processo só pode executar depois de sua chegada.
- A simulação termina quando todos os processos concluem.

Represente o tempo por uma variável numérica. As unidades de tempo são fictícias: avance essa variável conforme a execução simulada, sem realizar esperas reais.

### Ordem dos eventos e desempates

No instante `t`, registrar a execução que terminou em `t`, identificar conclusões e admitir os processos que chegam em `t` antes da próxima decisão de escalonamento.

Quando não houver uma regra específica do algoritmo, desempatar pelo menor instante de chegada e, depois, pelo ID em ordem lexicográfica. Para evitar ambiguidades, utilizar IDs como `P01`, `P02` e `P03`.

Os intervalos de execução seguem a convenção `[inicio, fim)`: um trecho de 2 a 5 representa três unidades de CPU.

## 4. Regras específicas dos algoritmos

### 4.1. FCFS / FIFO

- Manter uma fila pela ordem de chegada.
- O processo escolhido executa até concluir.
- Novas chegadas não interrompem o processo atual.

### 4.2. SJF

- Quando a CPU ficar disponível, escolher o processo pronto com o menor tempo total de CPU.
- O processo escolhido executa até concluir.
- Processos que ainda não chegaram não participam da escolha.

### 4.3. SRTF

- Reavaliar a escolha a cada chegada e conclusão.
- Executar o processo pronto com menor tempo restante.
- Interromper o atual apenas se outro processo apresentar tempo restante estritamente menor.
- Em caso de empate com o processo atual, mantê-lo na CPU. Se a CPU estiver disponível, aplicar o desempate geral.

### 4.4. Round Robin

- Utilizar uma fila FIFO de processos prontos.
- Executar cada processo por até `quantum` unidades, ou até concluir.
- Inserir novas chegadas no final da fila, sem interromper a fatia atual.
- Quando o quantum terminar, reinserir o processo não concluído no final da fila.
- Processos que chegam exatamente ao fim do quantum entram antes da reinserção do processo interrompido.

### 4.5. MLFQ: versão didática

Utilizar três filas, numeradas de 0 a 2, com prioridade decrescente e quanta definidos em `quantuns`.

- Novos processos entram no final da fila 0.
- Sempre executar a fila de maior prioridade que não esteja vazia.
- Dentro de cada fila, utilizar Round Robin.
- O processo que consumir o quantum completo sem concluir desce uma fila.
- Na fila 2, permanece na mesma fila e retorna ao final ao esgotar seu quantum.
- Uma chegada na fila 0 interrompe um processo que esteja executando em uma fila inferior.
- Nesse caso, o processo interrompido permanece na frente de sua fila e conserva o saldo do quantum. A interrupção não reinicia sua cota.
- Uma chegada à mesma fila não interrompe o processo atual.
- A cada `boost` unidades, nos instantes positivos múltiplos desse valor, mover todos os processos prontos, incluindo o atual, para a fila 0 e reiniciar suas cotas.
- No boost, ordenar os processos pelo instante de chegada e pelo ID.
- Se boost, chegada e fim de quantum coincidirem, processar a conclusão, admitir as chegadas e aplicar o boost. O boost prevalece sobre o rebaixamento.

Exemplo de configuração: quanta `[2, 4, 8]` e boost a cada 20 unidades.

### 4.6. CFS: versão didática

Cada processo possui um peso positivo e um contador `vruntime`. O peso padrão é 1024.

- Reavaliar a escolha a cada unidade de tempo.
- Selecionar o processo pronto com menor `vruntime`.
- Atualizar o contador do processo que executou uma unidade:

```text
vruntime = vruntime + 1024 / peso
```

- Inicializar os primeiros processos com `vruntime = 0`.
- Ao admitir novos processos, inicializar seus contadores com o menor `vruntime` entre os processos já presentes, incluindo o atual. Se não houver nenhum, utilizar o último valor de referência registrado; inicialmente, zero.
- Ao fim de cada unidade, atualizar o valor de referência para o menor `vruntime` entre os processos ainda presentes. Se nenhum permanecer, conservar o `vruntime` do último processo concluído.
- Admitir todas as chegadas de um mesmo instante com o mesmo valor inicial.
- Em empate de `vruntime`, escolher quem executou há mais tempo. Processos que nunca executaram têm preferência; persistindo o empate, aplicar chegada e ID.
- Comparar contadores com tolerância numérica de `1e-9`.

Não é obrigatório implementar árvore rubro-negra. Uma lista ou outra estrutura pode ser usada, desde que a escolha respeite as regras. O README deve explicar a estrutura escolhida e as simplificações desse modelo.

## 5. Interface padronizada

A linguagem é livre, mas o programa deve **ler um objeto JSON pela entrada padrão** e **escrever um objeto JSON pela saída padrão**. Isso permite executar os simuladores com os mesmos arquivos de entrada.

Exemplos de execução:

```bash
python simulador.py < entrada.json > resultado.json
```

```bash
node simulador.js < entrada.json > resultado.json
```

Outras linguagens devem oferecer uma execução equivalente, documentada no README. Não exigir interação por menus para executar a simulação.

### 5.1. Entrada

```json
{
  "algoritmo": "RR",
  "configuracao": {
    "quantum": 2
  },
  "processos": [
    {"id": "P01", "chegada": 0, "cpu": 8},
    {"id": "P02", "chegada": 1, "cpu": 4},
    {"id": "P03", "chegada": 2, "cpu": 2},
    {"id": "P04", "chegada": 4, "cpu": 1}
  ]
}
```

| Campo | Regra |
|---|---|
| `algoritmo` | Um dos identificadores da seção 2 implementados pela dupla. |
| `configuracao` | Objeto com os parâmetros do algoritmo; `{}` para FCFS, SJF, SRTF e CFS. |
| `processos` | Lista não vazia de processos. |
| `id` | String não vazia e única. O valor `OCIOSO` é reservado. |
| `chegada` | Inteiro maior ou igual a zero. |
| `cpu` | Inteiro estritamente positivo. |
| `peso` | Inteiro positivo, utilizado pelo CFS; se omitido, assumir 1024. |

Para RR, exigir `{"quantum": 2}` ou outro inteiro positivo. Para MLFQ, exigir uma configuração como `{"quantuns": [2, 4, 8], "boost": 20}`, com três quanta inteiros positivos em ordem não decrescente e boost inteiro positivo.

A ordem dos processos no arquivo não deve substituir as regras de chegada e desempate. Campos conhecidos, mas não utilizados pelo algoritmo, como `peso` fora do CFS, devem ser ignorados.

### 5.2. Saída

A saída deve conter o algoritmo, a linha do tempo, as métricas individuais e as médias. O exemplo completo abaixo corresponde a **FCFS**, com P01 chegando em 0 e CPU 3, e P02 chegando em 1 e CPU 2:

```json
{
  "algoritmo": "FCFS",
  "linha_do_tempo": [
    {"processo": "P01", "inicio": 0, "fim": 3},
    {"processo": "P02", "inicio": 3, "fim": 5}
  ],
  "processos": [
    {
      "id": "P01",
      "primeira_execucao": 0,
      "conclusao": 3,
      "turnaround": 3,
      "espera": 0,
      "resposta": 0
    },
    {
      "id": "P02",
      "primeira_execucao": 3,
      "conclusao": 5,
      "turnaround": 4,
      "espera": 2,
      "resposta": 2
    }
  ],
  "medias": {
    "turnaround": 3.5,
    "espera": 1.0,
    "resposta": 1.0
  }
}
```

- Registrar intervalos ociosos com `"processo": "OCIOSO"`, incluindo o intervalo inicial quando a primeira chegada ocorrer depois de zero.
- Os intervalos devem cobrir a simulação inteira, sem sobreposições nem lacunas, e ter duração positiva.
- Unir intervalos adjacentes do mesmo processo. Essa normalização também se aplica quando o processo recebe quanta consecutivos sem outro processo executar entre eles.
- Ordenar as métricas individuais pelo ID.
- Emitir números JSON, não strings com unidades. Preservar precisão das médias; a correção aceitará diferença absoluta de até `1e-6`.
- Não imprimir menus, mensagens de depuração ou diagramas na saída padrão. Mensagens auxiliares podem ir para a saída de erro.
- Uma interface gráfica ou visualização de Gantt é opcional e não substitui a interface JSON.

### 5.3. Entradas inválidas

Validar os campos obrigatórios, os tipos, os valores e os parâmetros do algoritmo. Em caso de entrada inválida ou algoritmo não implementado, retornar um JSON como:

```json
{"erro": "O tempo de CPU deve ser um inteiro positivo."}
```

Encerrar com código diferente de zero e não retornar uma simulação parcial.

## 6. Métricas obrigatórias

Para cada processo, calcular:

```text
turnaround = conclusão − chegada
espera = turnaround − tempo de CPU
resposta = primeira execução − chegada
```

A fórmula de espera acima vale para o modelo deste trabalho, sem E/S e com custo zero de troca de contexto. Calcular também a média de cada métrica entre todos os processos.

## 7. Experimentos e análise

### Experimento 1: conjunto comum

Executar FCFS e o segundo algoritmo sobre os processos da entrada da seção 5.1. Alterar apenas o algoritmo e sua configuração. Para RR, usar quantum 2; para MLFQ, quanta `[2, 4, 8]` e boost 20; para CFS, pesos iguais a 1024.

Apresentar, para cada algoritmo:

1. Diagrama de Gantt, desenhado ou gerado a partir da linha do tempo.
2. Tabela de métricas por processo e respectivas médias.
3. Explicação das decisões de escalonamento, incluindo preempções quando existirem.
4. Comparação com FCFS: quais métricas melhoraram ou pioraram nesse conjunto?

### Experimento 2: casos elaborados pela dupla

Criar pelo menos três entradas adicionais e justificar o objetivo de cada uma:

- Um caso com CPU ociosa entre chegadas.
- Um caso com processos de durações diferentes e chegadas durante uma execução.
- Um caso com empates que exercite as regras de desempate.

Conferir manualmente pelo menos um desses casos. Todos devem ser executados nos dois algoritmos implementados.

### Experimento 3: característica do segundo algoritmo

| Algoritmo | Análise solicitada |
|---|---|
| SJF | Mostrar como um processo longo pode esperar enquanto processos curtos são escolhidos. Explicar quando chegadas contínuas podem causar starvation. |
| SRTF | Mostrar uma chegada que interrompe o processo atual e outra que não o interrompe. |
| RR | Comparar quanta 1, 2 e 4 usando os mesmos processos. Discutir resposta e frequência de alternância. |
| MLFQ | Mostrar um rebaixamento, uma interrupção por prioridade e um boost. Usar processos longos o suficiente para que os três eventos ocorram. |
| CFS | Comparar pesos iguais e diferentes. Criar um intervalo em que dois processos longos estejam prontos simultaneamente e comparar o tempo de CPU recebido nesse intervalo. |

No CFS, não comparar apenas o tempo total de CPU recebido até todos terminarem: esse total é fixado pela entrada. Comparar a distribuição durante a disputa pela CPU. Nos demais algoritmos, evitar afirmar que um resultado obtido em um único conjunto prova superioridade em qualquer situação.

## 8. Entrega

Entregar o código-fonte e os seguintes materiais:

- `README.md` com identificação da dupla, algoritmos implementados, requisitos, instruções de execução e estruturas de dados utilizadas.
- Arquivos JSON de entrada e de saída dos experimentos.
- Relatório curto com diagramas de Gantt, métricas, conferência manual e respostas da seção 7.

O código deve organizar separadamente, sempre que possível, a leitura e validação, a política de escalonamento e o cálculo das métricas. O arquivo de entrada não deve exigir alterações no código para ser utilizado.

Na demonstração, a dupla deverá executar uma entrada fornecida pelo professor e explicar uma decisão do algoritmo e um cálculo de métrica. Ambos os integrantes devem compreender a implementação.

## 9. Critérios de avaliação

- Correção das decisões de escalonamento e das regras de desempate.
- Cumprimento do padrão de entrada e saída.
- Correção da linha do tempo e das métricas.
- Qualidade dos experimentos e da análise comparativa.
- Organização, clareza do código e documentação.
- Capacidade de demonstrar e explicar a solução.



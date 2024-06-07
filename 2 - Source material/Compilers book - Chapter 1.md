2024-05-26 01:55

Link: Compilers book

# Compilers book - Chapter 1

### Processadores de linguagens
- um compilador traduz uma linguagem de facil leitura e escrita por humanos para a linguagem de maquina
  
- em linhas gerais a tarefa base do compilador eh receber um programa fonte e transforma-lo em um programa objeto, comunicando ao usuario possiveis erros no programa fonte
  
- um interpretador eh um tipo de processador de linguagem, onde o mesmo aceita o programa fonte e as entradas e gera uma saida, que e o resultado da computacao
- ![[Pasted image 20240526201402.png]]
- compiladores geralmente geram programas mais eficientes que interpretadores por diversos motivos, um deles sendo o fato de um compilador produzir codigo que roda diretamente no hardware, sem o overhead de um interpretador rodando ao mesmo tempo que busca gerar as saidas do programa.
  
- compiladores geram uma linguagem simbolica intermediaria, mais conhecida como assembly ao inves de gerar diretamente o codigo de maquina por ser mais de depurar e mais facil de ser gerada

- o *assembler* recebe o assembly, que consiste em instrucoes universais e converte para instrucoes especificas da maquina onde esta sendo executado, para que esta possa rodar o programa com instrucoes. ele tambem eh responsavel por mapear as variaveis para seus enderecos de memoria, para que assim seus valores possam ser usados.

### Compiladores
- As atividades de um compilador podem ser divididas em duas partes: analise e sintese

- A analise eh resposavel por criar uma representacao intermediaria do programa, atraves de um desmembramento do programa em menores e obrigando uma estrutura gramatical sobre elas. Faz tanto a analise sintatica quanto semantica 

- A tabela de simbolos tambem eh construida na parte da analise, que mais tarde eh encaminhada a parte de sintese junto com a representacao intermediaria.

- Ja a sintese constroi o programa objeto atraves da representacao intermediaria e da tabela de simbolos fornecida pelo passo anterior.

- Analisador lexico recebe um sequencia de caracteres e produz lexemas no formato:
  `<nome do token, valor do token>`
  `Nome do token`: simbolo abstrato
  `Valor do token`: aponta para uma entrada na tabela de simbolos

- A analise sintatica eh responsavel por criar a arvore sintatica do programa, onde cada no interior tem como seus filhos os argumentos de uma operacao. As fases seguintes de um compilador usarao a saida deste processo para gerar o programa objeto

- Na fase de analise sintatica a arvore sintatica passa por uma verificacao para descobrir se todas as operacoes e argumentos aparacem em uma ordem que sigam as definidas pela liguagem. Nessa fase eh onde acontecem coercoes de tipo.

- O proximo passo eh a geracao de codigo intermediario, onde uma vez que garantimos que o a sintaxe e semantica da linguagem sao respeitadas, eh produzido um codigo intermediario, para uma maquina abstrata. Esse codigo tem a funcao de ser facilmente traduzido para uma liguagem de maquina real, geralmente para a versao do CPU da maquina host do processo de compilacao.i
# Projeto 2

Este repositório contém o modelo de um sistema de transporte de carga desenvolvido usando o UPPAAL. O sistema consiste em uma via circular dividida em 8 seções e 3 veículos (trens) que transportam cargas entre estas seções. Este documento fornece uma descrição detalhada do sistema, dos autômatos temporizados utilizados e das propriedades verificadas.

## Descrição Geral

O sistema modela um transporte de carga onde:

* Via Circular: Composta por 8 seções numeradas de 1 a 8.
* Veículos (Trens): Três trens que se movem ao longo das seções.
* Estacionamento: Local onde os trens iniciam e onde retornam para reabastecimento.
* Semáforos: Controlam a entrada dos trens nas seções para garantir que apenas um trem esteja em uma seção por vez, exceto entre as seções 2 e 3, que juntas não podem ter mais de um trem.
* Regras de Operação:
  * Deslocamento: O tempo para se mover entre seções é de 10 unidades de tempo.
  * Carregamento/Descarregamento: Ao entrar em uma nova seção, o trem deve parar para carregar/descarregar por pelo menos 25 unidades de tempo.
  * Autonomia dos Trens: O trem n deve parar para reabastecer a cada n voltas completas na via.
 
![Sistema modelado](/autom_trens.png)
 
## Detalhes dos Autômatos

### Autômato do Trem

Estados Principais:
* `estacionamento`: Trem parado no estacionamento.
* `sec_n_descarga`: Trem realizando carregamento/descarregamento na seção `n`.
* `sec_n_saida`: Trem se deslocando para a próxima seção.

Variáveis:
* `time`: Relógio local para controlar tempos de deslocamento e carregamento.
* `nvoltas`: Contador de voltas realizadas pelo trem.

Transições:
* Deslocamento entre Seções: Ocorre se `time >= 10` e o semáforo da próxima seção permite.
* Carregamento/Descarregamento: O trem permanece no estado de carregamento por pelo menos 25 unidades de tempo.
* Reabastecimento: Após completar `n` voltas, o trem retorna ao estacionamento para reabastecer.

### Semáforos da Seções

Os semáforos são modelados para controlar o acesso dos trens às seções:

* Semáforos Gerais: Controlam as seções 1, 4, 5, 6, 7 e 8.
* Semáforo Especial (Seções 2 e 3):
  * O semáforo da seção 3 está **sempre** verde, e o da seção 2 só retorna a ficar verde quando o trem sai da seção 3.


Variáveis:
* `sem[n]`: Indica o estado do semáforo da seção `n` (0 para verde, 1 para vermelho).

Transições:
* O semáforo muda para vermelho quando um trem entra na seção e para verde quando o trem sai.

## Propriedades Verificadas

As seguintes propriedades foram especificadas e verificadas usando o UPPAAL: 

1. Ausência de deadlocks: <br>
  `A[] not deadlock`: O sistema é livre de deadlocks.
2. Controle de voltas dos trens: <br>
  `A[] not (trem1.nvoltas > trem1.n)`: O trem 1 não excede sua autonomia.
   Verificação similar para os outros trens.
3. Posição dos trens sem autonomia: <br>
  `A[](trem1.nvoltas == trem1.n) imply trem1.location == trem1.estacionamento || trem1.location == trem1.sec_1_descarga`: O trem sem autonomia está na seção 1 ou no estacionamento.
  Verificação similar para os outros trens.
4. Segurança nas seções: <br>
  `A[] not (((trem1.sec_5...) && (trem2.sec_5...)) ||
            ((trem1.sec_5...) && (trem3.sec_5...)) ||
            ((trem2.sec_5...) && (trem3.sec_5...)))`: Nunca há mais de um trem na seção 5.
   Verificação similar para as outras seções. A verificação das seções 2 e 3 é parecida, porém considera as duas seções como uma só.
5. Tempo mínimo para completar uma volta: <br>
  `A[] (trem1.nvoltas > 0 || trem2.nvoltas > 0 || trem3.nvoltas > 0) imply tempo_abs >= 280`: Uma volta completa não pode ser realizada em menos de 280 unidades de tempo (35 para cada seção: locomoção + carga/descarga).
   



























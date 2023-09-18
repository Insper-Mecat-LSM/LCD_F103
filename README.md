Autor: [Kaique Dognani](https://github.com/kaiqued)

# Criar um caractere novo para o display LCD
### Objetivo:
O objetivo deste tutorial é criar um novo caractere para o display LCD, inserí-lo na CGRAM (memória interna que guarda novos caracteres) e mostrar este caractere no display.

### Materiais:
- Nucleo F103RB
- Keil Studio Cloud
- LCD Keypad Shield

## 1. Primeiros passos...
Para poder escrever no display precisamos primeiro entender o display que estamos trabalhando. Podemos acessar o [datasheet](https://www.sparkfun.com/datasheets/LCD/HD44780.pdf) deste para entender um pouco melhor seu funionamento, mas para fins didádicos vou tentar trazer os pontos mais importantes para este tutorial. 

### O display:
Como podemos observar na imagem abaixo, o display dispõe de 2 linhas e 16 colunas onde podemos "escrever" letras por exemplo.

![image](https://github.com/Insper-Mecat-LSM/LCD_F103/assets/26482377/6e1381cf-fe86-4184-a0e0-b3f0ced419d1)

Na página 17 do [datasheet](https://www.sparkfun.com/datasheets/LCD/HD44780.pdf) podemos verificar todos os dígitos possíveis de serem escritos, nos títulos das colunas temos os MSB (Most Significant Bits) e nas linhas os LSB (Least Significant Bits). Estes dígitos ficam guardados na CGROM do dispositivo, que é a memória de caracteres gráficos de somente leitura. No entanto, essa memória não permite a criação de novos caracteres personalizados. Para isso, precisamos utilizar a CGRAM, que é uma área de memória do display LCD reservada para armazenar até 8 caracteres personalizados de 5x8 pixels cada. Na mesma página 17 do Datasheet você pode verificar que a primeira coluna está vazia, é ali que está localizada a CGRAM onde vamos armazenar nossos dígitos!

Precisamos também entender os pinos que vamos utilizar deste display, que no caso serão 6:
1. RS, Register Select. Este pino é responsável por comunicar ao display se vamos enviar um comando/instrução ou se vamos escrever no display. Este pino está conectado na porta D8 da Nucleo;
2. D4 - D5 - D6 - D7, pinos de dados. Responsáveis por enviar o comando/instrução/texto para o display. Estes pinos estão conectados nas portas D4, D5, D6 e D7 da Nucleo;
3. E, Enable. Responsável por dizer quando uma operação começa ou finaliza. Este pino está conectado na porta D9 da Nucleo;

É importante ressaltar aqui que no shield que estamos utilizando temos o pino R/W soldado ao GND, portanto podemos apenas fazer operações de leitura!


### A comunicação:
Primeiro vamos entender como a Nucleo vai se comunicar com o display. Sempre que vamos iniciar uma comunicação precisamos ativar os pinos de dados conforme necessário e na sequência ligar o pino de enable por 400ns, desta forma estamos avisando o display que ele pode ler os dados que estamos enviando. Para escrever a letra "H" no display, por exemplo, podemos seguir os seguintes passos:


1. Ligar o pino RS;
2. Ligar os pinos de dados para escrever a primeira metade (Os dígitos mais significativos) da letra "H";
![image](https://github.com/Insper-Mecat-LSM/LCD_F103/assets/26482377/c8d42374-7a04-4038-aa9e-2fa5f17cc00f)

  Até o momento temos então:
| RS | D4 | D5 | D6 | D7 |
| :-: | :-: | :-: | :-: | :-: |
| 1 | 0 | 1 | 0 | 0 |

3. Ligar o pino de enable;
4. Aguardar 400ns;
5. Desligar o pino de enable;
6. Aguardar 400ns;
7. Ligar os pinos de dados para escrever a segunda metade (Os dígitos menos significativos) da letra "H";

  Novamente agora teremos:
| RS | D4 | D5 | D6 | D7 |
| :-: | :-: | :-: | :-: | :-: |
| 1 | 1 | 0 | 0 | 0 |

8. Ligar o pino de enable;
9. Aguardar 400ns;
10. Desligar o pino de enable;
11. Aguardar 400ns;



## 2. Como transformamos isto em código?
### Preparando o ambiente de desenvolvimento:
Primeiro vamos criar um novo projeto no Keil Studio Cloud utilizando o exemplo Blink para Mbed 2:

![image](https://github.com/Insper-Mecat-LSM/LCD_F103/assets/26482377/50528c2c-dd3a-4c19-9882-bed10549be49)

### Includes e Definição dos pinos:
``` C++
#include "mbed.h"

DigitalOut RS(D8);              // Saída digital para o pino de RS
DigitalOut E(D9);               // Saída digital para o pino de Enable
BusOut dados(D4, D5, D6, D7);   // Saídas digitais de comunicação com o display
```

### Funções para simplificar:
Analisando o passo a passo da comunicação, verificamos que os passos 3, 4, 5, 6 e 8, 9, 10, 11 são exatatamente iguais, repetir estas instruções diversas vezes no código pode torná-lo longo e de difícil leitura. Para solucionar isto podemos criar uma função que simplifica este passo:
``` C++
void toggle_enable(void){
    E = 1;
    wait(0.000040f);
    E = 0;
    wait(0.000040f);
}
```

Podemos também agora criar duas funções que vão nos auxiliar no envio de comandos e texto para 

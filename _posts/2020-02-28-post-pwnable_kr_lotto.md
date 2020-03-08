---
title: "pwnable.kr - Toddler's bottle #12 lotto"
date: 2020-02-28
categories:
  - Information Security
tags:
  - pwnable.kr
---

[pwnable.kr][pwnable.kr] Toddler's bottle의 #12 lotto 문제 풀이

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <fcntl.h>

unsigned char submit[6];

void play(){

        int i;
        printf("Submit your 6 lotto bytes : ");
        fflush(stdout);

        int r;
        r = read(0, submit, 6);

        printf("Lotto Start!\n");
        //sleep(1);

        // generate lotto numbers
        int fd = open("/dev/urandom", O_RDONLY);
        if(fd==-1){
                printf("error. tell admin\n");
                exit(-1);
        }
        unsigned char lotto[6];
        if(read(fd, lotto, 6) != 6){
                printf("error2. tell admin\n");
                exit(-1);
        }
        for(i=0; i<6; i++){
                lotto[i] = (lotto[i] % 45) + 1;         // 1 ~ 45
        }
        close(fd);

        // calculate lotto score
        int match = 0, j = 0;
        for(i=0; i<6; i++){
                for(j=0; j<6; j++){
                        if(lotto[i] == submit[j]){
                                match++;
                        }
                }
        }

        // win!
        if(match == 6){
                system("/bin/cat flag");
        }
        else{
                printf("bad luck...\n");
        }

}

void help(){
        printf("- nLotto Rule -\n");
        printf("nlotto is consisted with 6 random natural numbers less than 46\n");
        printf("your goal is to match lotto numbers as many as you can\n");
        printf("if you win lottery for *1st place*, you will get reward\n");
        printf("for more details, follow the link below\n");
        printf("http://www.nlotto.co.kr/counsel.do?method=playerGuide#buying_guide01\n\n");
        printf("mathematical chance to win this game is known to be 1/8145060.\n");
}

int main(int argc, char* argv[]){

        // menu
        unsigned int menu;

        while(1){

                printf("- Select Menu -\n");
                printf("1. Play Lotto\n");
                printf("2. Help\n");
                printf("3. Exit\n");

                scanf("%d", &menu);

                switch(menu){
                        case 1:
                                play();
                                break;
                        case 2:
                                help();
                                break;
                        case 3:
                                printf("bye\n");
                                return 0;
                        default:
                                printf("invalid menu\n");
                                break;
                }
        }
        return 0;
}
```

처음에는 /dev/urandom 의 취약점을 찾아야 하는 문제인가? 하고 인터넷을 뒤져봤지만 별로 소득은 없었다.

그러다 소스코드를 유심히 살펴 보니 번호를 확인하는 루프가 더블루프네?

6개중에 하나만 맞추면 6개 다 맞출수 있네?

가능한 번호가 1-45니까 ascii table에 있는것중 32에 해당하는 !를 연속으로 입력해 봤다.

여러번 하다보니 결국엔 성공. 약간 어이없는 문제였음.

~~~
...
- Select Menu -
1. Play Lotto
2. Help
3. Exit
1
Submit your 6 lotto bytes : !!!!!!
Lotto Start!
<tag output>
...
~~~

[pwnable.kr]: https://pwnable.kr

<!-- sorry mom... I FORGOT to check duplicate numbers... :( -->
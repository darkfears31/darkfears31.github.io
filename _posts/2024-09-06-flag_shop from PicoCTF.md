---
title: "flag_shop from PicoCTF"
categories: [PicoCTF]
tags: [PicoCTF]
---
# flag_shop from PicoCTF
[page](https://play.picoctf.org/practice/challenge/49?category=5&difficulty=2&page=1&search=&solved=0)
> Description
There's a flag shop selling stuff, can you buy a flag? [Source](https://jupiter.challenges.picoctf.org/static/64e724ad327f83ad833d9c6baa072b1f/store.c). Connect with `nc jupiter.challenges.picoctf.org 4906`.

>Hints
>Two's compliment can do some weird things when numbers get really big!

When we read `source` code we see this:
```
#include <stdio.h>
#include <stdlib.h>
int main()
{
    setbuf(stdout, NULL);
    int con;
    con = 0;
    int account_balance = 1100;
    while(con == 0){
        
        printf("Welcome to the flag exchange\n");
        printf("We sell flags\n");

        printf("\n1. Check Account Balance\n");
        printf("\n2. Buy Flags\n");
        printf("\n3. Exit\n");
        int menu;
        printf("\n Enter a menu selection\n");
        fflush(stdin);
        scanf("%d", &menu);
        if(menu == 1){
            printf("\n\n\n Balance: %d \n\n\n", account_balance);
        }
        else if(menu == 2){
            printf("Currently for sale\n");
            printf("1. Defintely not the flag Flag\n");
            printf("2. 1337 Flag\n");
            int auction_choice;
            fflush(stdin);
            scanf("%d", &auction_choice);
            if(auction_choice == 1){
                printf("These knockoff Flags cost 900 each, enter desired quantity\n");
                
                int number_flags = 0;
                fflush(stdin);
                scanf("%d", &number_flags);
                if(number_flags > 0){
                    int total_cost = 0;
                    total_cost = 900*number_flags;
                    printf("\nThe final cost is: %d\n", total_cost);
                    if(total_cost <= account_balance){
                        account_balance = account_balance - total_cost;
                        printf("\nYour current balance after transaction: %d\n\n", account_balance);
                    }
                    else{
                        printf("Not enough funds to complete purchase\n");
                    }
                                    
                    
                }
                    
                    
                    
                
            }
            else if(auction_choice == 2){
                printf("1337 flags cost 100000 dollars, and we only have 1 in stock\n");
                printf("Enter 1 to buy one");
                int bid = 0;
                fflush(stdin);
                scanf("%d", &bid);
                
                if(bid == 1){
                    
                    if(account_balance > 100000){
                        FILE *f = fopen("flag.txt", "r");
                        if(f == NULL){

                            printf("flag not found: please run this on the server\n");
                            exit(0);
                        }
                        char buf[64];
                        fgets(buf, 63, f);
                        printf("YOUR FLAG IS: %s\n", buf);
                        }
                    
                    else{
                        printf("\nNot enough funds for transaction\n\n\n");
                    }}

            }
        }
        else{
            con = 1;
        }

    }
    return 0;
}
```
There is one thing wrong here: 
```
int number_flags = 0;
                fflush(stdin);
                scanf("%d", &number_flags);
                if(number_flags > 0){
                    int total_cost = 0;
                    total_cost = 900*number_flags;
                    printf("\nThe final cost is: %d\n", total_cost);
                    if(total_cost <= account_balance){
                        account_balance = account_balance - total_cost;
                        printf("\nYour current balance after transaction: %d\n\n", account_balance);
```
If we buy `definetely not Flag flag` after transaction we our balance rises, because of this:
```
account_balance = account_balance - total_cost
```
Notice that `total_cost` is declared as an `int`. This is specifically a _Signed Integer_, which has a range between `-2,147,483,648` to `2,147,483,647`. Signed integers use the first bit to store whether it is negative or positive, `0` indicating positive and `1` indicating negative. What happens if you add `1` to `2,147,483,647` and store the result in a signed integer? Well the first bit goes from `0` to `1`, meaning that the number is now negative! In fact, due to the way [Two's Complement](https://en.wikipedia.org/wiki/Two%27s_complement), the method used to represent negative numbers in binary, works, it actually wraps around to the most negative integer: `-2,147,483,648`.

This is what is known as an Integer Overflow. We can use this to overflow the `total_cost` variable and increase our account balance. We need a `total_cost` that is a large negative number, but not too large that it also overflows our `account_balance`. `2900000` works nicely as the number of fake flags to buy. this will become negative number and with basic math this will become positive and add to our `account_balance`. After that we can buy the real flag.
Flag is: 
**picoCTF{m0n3y_bag5_b9f469b5}**

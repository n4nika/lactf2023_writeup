# string-cheese (rev) by aplet123

### Description:
```
rev/string-cheese
aplet123
644 solves / 112 points

I'm something of a cheese connoisseur myself. If you can guess my favorite flavor of string cheese, I'll even give you a flag. Of course, since I'm lazy and socially inept, I slapped together a program to do the verification for me.

Connect to my service at nc lac.tf 31131

Note: The attached binary is the exact same as the one executing on the remote server.
```

We are given a binary and a netcat port. When we execute binary we see:
```
What's my favorite flavor of string cheese?
```
and are prompted for an input. If the input is wrong, the binary exits.

### Solve:

First I threw the binary into ghidra and had a look at the decompiled "main":
```
undefined8 main(void)

{
  int iVar1;
  size_t sVar2;
  char local_108 [256];
  
  printf("What\'s my favorite flavor of string cheese? ");
  fflush(stdout);
  fgets(local_108,0x100,stdin);
  sVar2 = strcspn(local_108,"\n");
  local_108[sVar2] = '\0';
  iVar1 = strcmp(local_108,"blueberry");
  if (iVar1 == 0) {
    puts("...how did you know? That isn\'t even a real flavor...");
    puts("Well I guess I should give you the flag now...");
    print_flag();
  }
  else {
    puts("Hmm... I don\'t think that\'s quite it. Better luck next time!");
  }
  return 0;
}
```
hmm.. as we see there is a simple string-comparison with the hardcoded value "blueberry".
So I executed the programm remotely, entered the "password" and:
```
What's my favorite flavor of string cheese? blueberry
...how did you know? That isn't even a real flavor...
Well I guess I should give you the flag now...
lactf{d0n7_m4k3_fun_0f_my_t4st3_1n_ch33s3}
```
Nice and simple challenge :)

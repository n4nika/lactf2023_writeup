# chinese-lazy-theorem-1 (crypto) by joshuazhu17

### Description:
```
crypto/chinese-lazy-theorem-1
joshuazhu17
343 solves / 238 points

I heard about this cool theorem called the Chinese Remainder Theorem, but, uh... I'm feeling kinda tired right now.

nc lac.tf 31110
```

With the challenge we get a netcat port and a python script. When we execute it and play with it a little bit we get:
```
10662290236207282019263538410504816901540183950289460370214080222905782861627374056251804372336168549709470957558091508834785020797696386901583230930233547
10400792794453506822497021858150371734813592890041673243083813302444860941423466758138490583292186222475219483259176878900821324679812165425564213639620467
To quote Pete Bancini, "I'm tired."
I'll answer one modulus question, that's it.
What do you want?
1: Ask for a modulus
2: Guess my number
3: Exit
>> 1
Type your modulus here: 1337
39

What do you want?
1: Ask for a modulus
2: Guess my number
3: Exit
>> 2
Type your guess here: 1337
nope
```
But since we got the file let's have a look at that.

### Source Code:
```
#!/usr/local/bin/python3

from Crypto.Util.number import getPrime
from Crypto.Random.random import randint

p = getPrime(512)
q = getPrime(512)
n = p*q

target = randint(1, n)

used_oracle = False

print(p)
print(q)

print("To quote Pete Bancini, \"I'm tired.\"")
print("I'll answer one modulus question, that's it.")
while True:
    print("What do you want?")
    print("1: Ask for a modulus")
    print("2: Guess my number")
    print("3: Exit")
    response = input(">> ")

    if response == "1":
        if used_oracle:
            print("too lazy")
            print()
        else:
            modulus = input("Type your modulus here: ")
            modulus = int(modulus)
            if modulus <= 0:
                print("something positive pls")
                print()
            else:
                used_oracle = True
                print(target%modulus)
                print()
    elif response == "2":
        guess = input("Type your guess here: ")
        if int(guess) == target:
            with open("flag.txt", "r") as f:
                print(f.readline())
        else:
            print("nope")
        exit()
    else:
        print("bye")
        exit()
```
### Solution:

Looking at it, we see that we have to "guess" the value of `target`. What's interesting, is the option `1` to ask for a modulus.
In here, `target` is taken modulo with a number provided by us. Hm, how could we abuse this? Since there is no limitation on the size of our input, we could just provide a number 
greater or equal to `n` in order to get the "raw" value of `target`. So I just input greater and greater numbers and provided them until I got the flag. 
A more straightforward way would be to just calculate `n` (since `p` and `q` are given) and input that when asked for a modulus.

So when doing it the straightforward way (multiplying `p` and `q`) we get:
```
11696234827393828139279054378890265318175019032558379177266142073660615510104347437852663914728315863064505797839546198976639204890465349548592414606952763
10334414584883007998613268591963140340928046989003177641426135528191415874389755871175625934213238820818120457667415120042840085818931354798581616683528767
To quote Pete Bancini, "I'm tired."
I'll answer one modulus question, that's it.
What do you want?
1: Ask for a modulus
2: Guess my number
3: Exit
>> 1
Type your modulus here: 120873739788435369140402876025619539394516749491797593477307365490190492311701577685693283691074443306917255455183202090318464539840171489106622548696415130067444240686905216454227356443125338980332960685108164070728654400805011358052426966829315220078193510648802936427512203266010432966824054048219720633221
74063724978047396135929485186362790358896901226652583957312367503438123862469090425241739676164891467168167839384819409765189952353243587861284441817803194570956544645351518001806580718960626401449618669594528889167406989242285558372108154322655190397590870018115173284415791174610567654677419932046886488999

What do you want?
1: Ask for a modulus
2: Guess my number
3: Exit
>> 2
Type your guess here: 74063724978047396135929485186362790358896901226652583957312367503438123862469090425241739676164891467168167839384819409765189952353243587861284441817803194570956544645351518001806580718960626401449618669594528889167406989242285558372108154322655190397590870018115173284415791174610567654677419932046886488999
lactf{too_lazy_to_bound_the_modulus}
```
Was a quick and fun challenge, thanks!
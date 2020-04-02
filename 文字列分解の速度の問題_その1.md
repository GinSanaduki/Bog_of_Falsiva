# bashで文字列分解する時、cutやawkもいいけど、setの方が早い、けどreadが最強
# ・・・でのawkの書き方に相当問題を感じたので、書き直した。

https://qiita.com/hasegit/items/5be056d67347e1553f08  
bashで文字列分解する時、cutやawkもいいけど、setの方が早い、けどreadが最強 - Qiita  

```bash

# hosts
123.123.123.123         geeg1   # application server
123.123.123.124         geeg2   # web frontend server
123.123.123.125         geeg3   # super fabulous exciting backup server #1

```

```bash

# case_of_awk
while read line
do
  ip=$(awk '{print $1}' <<<${line})
  hostn=$(awk '{print $2}' <<<${line})
  comment=$(awk '{for(i=4;i<NF;i++){ printf("%s%s",$i,OFS=" ")}print $NF}' <<<${line})
  echo "[ip]${ip} [hostname]${hostn} [comment]${comment}"
done < hosts

```

・・・とあるが、そりゃあ、こんなループでは、500回回して28.268秒といわれても、違和感はない。

```

本当はawk内で完結させようと思ったのですが、知識及ばず…。

```

とのことなので、awkで書いた。  
gawkは言うに及ばず、mawkだろうがnawkだろうが、元祖awkだろうが平然と動く。  

```awk

#!/usr/local/bin/awk
# test.awk
# awk -f test.awk hosts

{
	split($0,Arrays,"#");
	Comment = "";
	for(i in Arrays){
		if(i > 1){
			Comment = Comment "#"Arrays[i];
		}
	}
	print "[ip]"$1" [hostname]"$2" [comment]"Comment;
	delete Arrays;
}

```

速度は・・・

```bash

$ time for i in {0..500}; do awk -f test.awk hosts >/dev/null; done

real    0m4.738s
user    0m0.328s
sys     0m3.828s

```

さすがにreadの3.207秒には勝てなかったが、awk内でEND句で500件ループしてバカスカ表示するなら、速いかも。

```awk

BEGIN{
	Cnt = 1;
}

{
	split($0,Arrays,"#");
	Comment = "";
	for(i in Arrays){
		if(i > 1){
			Comment = Comment "#"Arrays[i];
		}
	}
	ArraysPrint[Cnt] = "[ip]"$1" [hostname]"$2" [comment]"Comment;
	delete Arrays;
	Cnt++;
}

END{
	for (i = 1; i <= 500; i++){
		for(j in ArraysPrint){
			print ArraysPrint[j];
		}
	}
}

```

```bash

$ time awk -f test.awk hosts >/dev/null

real    0m0.011s
user    0m0.000s
sys     0m0.016s

```

これなら、readに圧勝しました。

# Phew, what a relief!


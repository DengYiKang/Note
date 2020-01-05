# Manacher

在时间复杂度为O(n)的情况下求解一个字符串的最长回文子串长度的问题。

```java
String s;//input
char[] str=new char[maxn*2+1];
int[] p=new int[maxn];
int manacher(){
    int i=1;
    for(char ch:s){
        str[i*2]=ch;
        str[i*2+1]='#';
    }
    str[0]='?',str[1]='#',str[i*2]='\0';
    int res=0, k=0, maxk=0;
    /*
    str[maxk] is not contained in the matched segment of center str[k]
    */
    for(int i=2; str[i]; i++){
        p[i]=i<maxk?min(maxk-i, p[2*k-i]):1;
        while(str[i-p[i]]==str[i+p[i]]) p[i]++;
        if(p[i]+i>maxk){
            k=i;
            maxk=i+p[i];
        }
        res=max(res, p[i]);
    }
    return res-1;
}
```


# AC自动机

```java
public class acAutomata{
    static int K = 26;

    class Vertex {
        //v1.next[c]=v2表示v1状态接受输入c后成功进入v2状态
        int[] next = new int[K];    
        //对应的串的个数
        int cnt = 0;
        //避免重复检查
        boolean vis = false;
		//父结点序号
        int p = -1;
		//最近一次接受的输入
        char pch;
		//fail指针
        int link = -1;
		//辅助用，辅助计算link，go[ch]表示该状态输入ch后转移到哪个状态
        int[] go = new int[K];

        public Vertex(int p, char ch) {
            this.p = p;
            this.pch = ch;
            Arrays.fill(next, -1);
            Arrays.fill(go, -1);
        }

        public Vertex() {
            this.p = -1;
            this.pch = '$';
            Arrays.fill(next, -1);
            Arrays.fill(go, -1);
        }
    }

    List<Vertex> nodes = new ArrayList<Vertex>() {{
        add(new Vertex());
    }};

    //把pattern插入到字典树
    void add_string(String s) {
        int v = 0;
        for (int i = 0; i < s.length(); i++) {
            int c = s.charAt(i) - 'a';
            if (nodes.get(v).next[c] == -1) {
                nodes.get(v).next[c] = nodes.size();
                nodes.add(new Vertex(v, s.charAt(i)));
            }
            v = nodes.get(v).next[c];
        }
        nodes.get(v).cnt++;
    }

    //link值，即fail指针
    int get_link(int v) {
        Vertex t = nodes.get(v);
        if (t.link == -1) {
            if (v == 0 || t.p == 0)
                t.link = 0;
            else
                t.link = go(get_link(t.p), t.pch);
        }
        return t.link;
    }

    //go(v,ch)进行状态转移
    int go(int v, char ch) {
        Vertex t = nodes.get(v);
        int c = ch - 'a';
        if (t.go[c] == -1) {
            if (t.next[c] != -1) {
                t.go[c] = t.next[c];
            } else {
                t.go[c] = v == 0 ? 0 : go(get_link(v), ch);
            }
        }
        return t.go[c];
    }
	
    //输出匹配的pattern个数
    int dfa(String s) {
        int status = 0, ans = 0;
        for (int i = 0; i < s.length(); i++) {
            int c = s.charAt(i) - 'a';
            //寻找转移后的状态
            while (nodes.get(status).next[c] == -1 && status != 0) {
                status = get_link(status);
            }
            if (status == 0) {
                status = nodes.get(status).next[c] == -1 ? 0 : nodes.get(status).next[c];
            } else {
                status = nodes.get(status).next[c];
            }
            int tmp = status;
            /*
            *当成功转移至非0的状态后，状态0到tmp状态对应的字符串是匹配的，
            *那么状态0到tmp.link的字符串也是匹配的，不断跳link
            */
            while (tmp != 0) {
                if (nodes.get(tmp).vis) break;
                //避免重复计算
                nodes.get(tmp).vis = true;
                ans += nodes.get(tmp).cnt;
                tmp = get_link(tmp);
            }
        }
        return ans;
    }
}
```


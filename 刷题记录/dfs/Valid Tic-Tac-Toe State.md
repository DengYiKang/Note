# Valid Tic-Tac-Toe State

### 问题

两个人在一个`3*3`的棋盘上下三子棋，现给一个棋盘状态，问在规则下行动，能否达到这个状态。

### 解决方案

#### 方法一：dfs

当时考虑的太复杂，在想如果有多个连成三子的棋子但却不相通的话那么是非法状态。但是这个棋盘是`3*3`的，因此不存在这种情况。dfs适用于更大的棋盘。

```java
class Solution {
    public boolean validTicTacToe(String[] board) {
        char[][] A=new char[3][];
        for(int i=0; i<A.length; i++){
            A[i]=board[i].toCharArray();
        }
        //用list来存储棋子位置，用于dfs遍历
        List<List<int[]>> pos=new ArrayList<>();
        pos.add(new ArrayList<int[]>());
        pos.add(new ArrayList<int[]>());
        int leave=0;
        for(int i=0; i<A.length; i++){
            for(int j=0; j<A[i].length; j++){
                if(A[i][j]=='X') {
                    pos.get(0).add(new int[]{i, j});
                    leave++;
                }
                if(A[i][j]=='O') {
                    pos.get(1).add(new int[]{i, j});
                    leave++;
                }
            }
        }
        char[][] B=new char[3][3];
        for(int i=0; i<3; i++){
            Arrays.fill(B[i], ' ');
        }
        return dfs(0, B, pos, leave);
    }
    public boolean dfs(int turn, char[][] A, List<List<int[]>> pos, int leave){
        //这个情况下，谁都没有取胜
        if(leave==0) return true;
        List<int[]> cur_pos=pos.get(turn);
        for(int i=0; i<cur_pos.size(); i++){
            int[] npos=cur_pos.get(i);
            int nx=npos[0];
            int ny=npos[1];
            if(A[nx][ny]!=' ') continue;
            A[nx][ny]=turn==0?'X':'O';
            leave--;
            if(isEnd(nx, ny, A)){
                //这种情况下，某一方取胜，且整个棋局与所给的一致
                if(leave==0) return true;
                else{
                    //某一方提前取胜，非法，注意回溯，不能直接return
                    leave++;
                    A[nx][ny]=' ';
                    continue;
                }
            }
            if(dfs(1-turn, A, pos, leave)) return true;
            A[nx][ny]=' ';
            leave++;
        }
        return false;
    }
    public boolean isEnd(int x, int y, char[][] A){
        boolean[] flag=new boolean[4];
        Arrays.fill(flag, true);
        for(int i=0; i<3; i++){
            if(A[i][y]!=A[x][y]) flag[0]=false;
            if(A[x][i]!=A[x][y]) flag[1]=false;
            if(x!=y||A[i][i]!=A[x][y]) flag[2]=false;
            if((x+y)!=2||A[i][2-i]!=A[x][y]) flag[3]=false;
        }
        return flag[0]||flag[1]||flag[2]||flag[3];
    }
}
```

#### 方法二：统计

只需要判断是否满足下列条件：

+ 所给的棋盘上没有两个同时赢的人
+ 第一个人走的步数=第二个走的（或+1）

```java
class Solution {
    public boolean validTicTacToe(String[] board) {
        char[][] A=new char[3][3];
        for(int i=0; i<A.length; i++){
            A[i]=board[i].toCharArray();
        }
        //是'X'就+1，否则-1，最后可以根据这个判断两个棋手下的步数之差
        int turn=0;
        //rows[]，columns[], diags, antiDiags与turn一样
        int[] rows=new int[3];
        int[] columns=new int[3];
        int diags=0;
        int antiDiags=0;
        boolean xWin=false;
        boolean oWin=false;
        for(int i=0; i<3; i++){
            for(int j=0; j<3; j++){
                if(A[i][j]=='X'){
                    turn++;
                    rows[i]++;
                    columns[j]++;
                    if(i==j) diags++;
                    if(i+j==2) antiDiags++;
                }
                else if(A[i][j]=='O'){
                    turn--;
                    rows[i]--;
                    columns[j]--;
                    if(i==j) diags--;
                    if(i+j==2) antiDiags--;
                }
            }
        }
        for(int i=0; i<3; i++){
            xWin=xWin||(rows[i]==3)||(columns[i]==3);
            oWin=oWin||(rows[i]==-3)||(columns[i]==-3);
        }
        xWin=xWin||(diags==3)||(antiDiags==3);
        oWin=oWin||(diags==-3)||(antiDiags==-3);
        if((turn==1&&xWin&&!oWin)||(turn==0&&!xWin&&oWin)) return true;
        if(!xWin&&!oWin&&(turn==1||turn==0)) return true;
        return false;
    }
}
```


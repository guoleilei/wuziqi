# wuziqi
import javax.sound.sampled.AudioInputStream;
import javax.swing.*;
import java.applet.AudioClip;
import java.awt.*;
import java.net.URL;
import java.util.Arrays;
import java.util.LinkedList;
import java.util.Random;

@SuppressWarnings("serial")
/*
 * panel 面板
 * 控制下棋和判断胜负及算法的类
 * 
 */
public class ChessPanel extends JPanel{
	private ImageIcon map;					//棋盘背景位图
  	private ImageIcon blackchess;			//黑子位图
  	private ImageIcon whitechess;			//白子位图
  	public int isChessOn [][];				//棋局
    protected boolean win = false;          // 是否已经分出胜负
    protected int win_bw;                   // 胜利棋色
    protected int deep = 3, weight = 7;    // 搜索的深度以及广度
    public int drawn_num = 110;           // 和棋步数
    int chess_num = 0;                      // 总落子数目
    public int[][] pre = new int[drawn_num + 1][2];    // 记录下棋点的x,y坐标   最多 (drawn_num + 1) 个
    public int sbw = 0;                          //玩家棋色黑色0，白色1
    public int bw = 0;                           // 当前应该下的棋色  0：黑色(默认)， 1：白色
  	  // 边界值,用于速度优化
    protected int x_max = 15, x_min = 0;
    protected int y_max = 15, y_min = 0;
    protected boolean able_flag = true;       // 是否选择禁手标志 0:无禁手  1:有禁手(默认
  	private int h;							//棋子长
 	private int w;							//棋子宽
 	private int insx;						//插入棋子的位置
 	private int insy;
 	private Point mousePoint;				//鼠标当前位置
 	private int winer;						//获胜方
    private boolean humanhuman=false;       //是否是人人对弈
 	private int plast=0;					//走了几步了，
 	public int BLACK_ONE;					//0表黑子
 	public int WHITE_ONE;					//1表白子
 	public int NONE_ONE;					//2表无子
 	public int N;							//棋盘边长
 
 	//-------声音
 	 String[] choics = { "put.wav", "win.wav","lost.wav" }; //声音文件名数组
 	 URL file1 = getClass().getResource(choics[0]); //落子声音文件
 	 URL file2 = getClass().getResource(choics[1]); //获胜声音文件
 	 URL file3 = getClass().getResource(choics[2]); //失败声音文件
 	 AudioClip soundPut = java.applet.Applet.newAudioClip(file1); //落子声音剪辑对象
 	 AudioClip soundWin = java.applet.Applet.newAudioClip(file2); //获胜声音剪辑对象
 	 AudioClip soundLost = java.applet.Applet.newAudioClip(file3); //失败声音剪辑对象
 	
 	public ChessPanel(){}
 	public ChessPanel(ImageIcon r_map,ImageIcon r_blackchess,ImageIcon r_whitechess) {
 		
 		N=15;
 		map=new ImageIcon();
 		blackchess=new ImageIcon();
 		whitechess=new ImageIcon();
  		map=r_map;
  		blackchess=r_blackchess;
  		whitechess=r_whitechess;
  		NONE_ONE=2;
  		BLACK_ONE=0;
  		WHITE_ONE=1;
  		winer=NONE_ONE;
   		isChessOn=new int[N][N];
    	h=blackchess.getIconHeight()*(N-1);
    	w=blackchess.getIconWidth()*(N-1);
    	insx=0;
    	insy=0;
    	mousePoint=new Point();
    	
    }
	
    public void reset(){							//重开一局
  		winer=NONE_ONE;
  		for(int i=0;i<N;i++)
  			for(int j=0;j<N;j++){
  				isChessOn[i][j]=NONE_ONE;
  			}
  		chess_num = 0;  
  		win = false; 
  		win_bw=2;
  		bw = 0;
  		x_max = 15; x_min = 0;
  	    y_max = 15;y_min = 0;
  		repaint();
    }
    public void showMousePos(Point p){				//调试用，显示鼠标位置
  	    int cw;
  	    cw=h/N;
  	    mousePoint.x=p.x/cw;
  	    mousePoint.y=p.y/cw;
  	    repaint();
    }
    public Point getPoint(int x,int y){
    	int cw;
  	    insx=x;
  	    insy=y;
  	    cw=h/N;
  	  Point r=new Point(x/cw,y/cw);
  	  return r;
    }
  public void gameOver(int r_winer){			//游戏胜负已分
  	winer=r_winer;
  }
  public void paint(Graphics g){				//整体布局
    super.paint(g);
    paintChessMap(g); 
    paintChess(g);
    if(winer==BLACK_ONE){
    	g.drawString(new String("游戏结束！黑棋获胜！"),500,200);
    	
    }
    else if(winer==WHITE_ONE){
    	g.drawString(new String("游戏结束！白棋获胜！"),500,200);
    }
  }
  private void paintChessMap(Graphics g){		//画棋盘
  	map.paintIcon(this,g,10,10);
  	int j;
    g.setColor(Color.BLACK);
    for(j=0;j<N;j++){							//画线
    	g.drawLine(h/N/2,h/N*j+h/N/2,w-w/N+(N%2)*(h/N/2),h/N*j+h/N/2);
    	g.drawLine(w/N*j+h/N/2,h/N/2,w/N*j+h/N/2,h-h/N+(N%2)*(h/N/2));
    }
    g.fillRect(w/N*7+h/N/2-3,h/N*7+h/N/2-3,6,6);//画5个黑方块
    g.fillRect(w/N*3+h/N/2-3,h/N*3+h/N/2-3,6,6);
    g.fillRect(w/N*11+h/N/2-3,h/N*3+h/N/2-3,6,6);
    g.fillRect(w/N*3+h/N/2-3,h/N*11+h/N/2-3,6,6);
    g.fillRect(w/N*11+h/N/2-3,h/N*11+h/N/2-3,6,6);
  }
  private void paintChess(Graphics g){			//画棋子
  		int i,j;
  		for(i=0;i<N;i++)
  			for(j=0;j<N;j++){
  				if(isChessOn[i][j]==BLACK_ONE){
  					blackchess.paintIcon(this,g,w/N*i,h/N*j);
  				}
  				else if(isChessOn[i][j]==WHITE_ONE){
  					whitechess.paintIcon(this,g,w/N*i,h/N*j);
  				}	
  			}
  }
  //-------------------------------下棋声音设置-------------------------------------------------
  
  //落子声音
  public void putVoice(){
		soundPut.play();     
  }
  //获胜声音
  public void winVoice(){
	   soundWin.play();
  }
  //失败声音
  public void lostVoice(){
	  soundLost.play();
  }
  
   //----------------------电脑下棋-------------------------------//
  public void  putOne(int bwf ) {  //bwf 棋色 0:黑色 1：白色
      int x, y, mx = -100000000;
      x = y = -1;
      // 搜索最优下棋点
      int[][] bests = getBests( bwf );
      for (int k = 0; k < bests.length; k++) {
          int i = bests[k][0];
          int j = bests[k][1];
          // 有成5,则直接下子,并退出循环..没有,则思考对方情况
          if (getType(i, j, bwf) == 1) {
              x = i;
              y = j;
              break;
          }
          if (getType(i, j,1 - bwf) == 1) {
              x = i;
              y = j;
              break;
          }
          // 预存当前边界值
          int temp1=x_min,temp2=x_max,temp3=y_min,temp4=y_max;
          // 预设己方下棋,并更新边界值
          isChessOn[i][j] = bwf;
          resetMaxMin(i,j);
          // 预测未来
          int t = findMin(-100000000, 100000000, deep);
          // 还原预设下棋位置以及边界值
          isChessOn[i][j] = 2;
          x_min=temp1;
          x_max=temp2;
          y_min=temp3;
          y_max=temp4;
          // 差距小于1000，50%概率随机选取
          //System.out.println("外       :" + i + "," + j + "  mx:" + mx + "  t:" + t);
          if (t - mx > 1000 || Math.abs(t - mx)<1000 && randomTest(3)) {
              x = i;
              y = j;
              mx = t;
              //System.out.println(i + "," + j + "  mx:" + mx + "  t:" + t);
          }
         
      }
      System.out.println("x="+x+",y="+y);
     // addChess(x,y,(bwf+1)%2,true);
     // repaint();
      int step=0;
		step++;
		System.out.println("step "+step+":-----------------------------------------------");
		for(int i=0;i<15;i++,System.out.print("\n"))
			for(int j=0;j<15;j++)
				{
					if(isChessOn[j][i]!=2)System.out.print(isChessOn[j][i]);
					else	System.out.print(isChessOn[j][i]);
				}	
  	// 判断是否已分胜负
   	boolean flag = haveWin(x, y, bwf);
       //记录
      update( x, y );
      repaint();
      // 重设边界值
      resetMaxMin(x,y);
     //  胜负已分
      if (flag) 
          wined(bwf);
      if (!flag && chess_num >= drawn_num) {
          win = true;
          String str = drawn_num + "步没分胜负,判和棋!";
          JOptionPane.showMessageDialog(null,str);
          return;
      }
         
  }
  
  //---------搜索当前搜索状态极大值--------------------------------//
  //alpha 祖先节点得到的当前最小最大值，用于alpha 剪枝
  //beta  祖先节点得到的当前最大最小值，用于beta 剪枝。
  //step  还要搜索的步数
  //return 当前搜索子树极大值
  protected int findMax(int alpha, int beta, int step) {
  	int max = alpha;
      if (step == 0) {
          return evaluate();
      }
      int[][] rt = getBests(1 - sbw);
      for (int i = 0; i < rt.length; i++) {
          int x = rt[i][0];
      	int y = rt[i][1];
      	if (getType(x, y, 1 - sbw) == 1)   //电脑可取胜
      		return 100 * ( getMark(1) + step*1000 );
          isChessOn[x][y] = 1 - sbw;
          // 预存当前边界值
          int temp1=x_min,temp2=x_max,temp3=y_min,temp4=y_max;
          resetMaxMin(x,y);
          int t = findMin(max, beta, step - 1);
          isChessOn[x][y] = 2;
          // 还原预设边界值
          x_min=temp1;
          x_max=temp2;
          y_min=temp3;
          y_max=temp4;
          if (t > max)
          	max = t;
          //beta 剪枝
          if (max >= beta) 
              return max;
      }
      return max;
  }
  

   //-----------------------搜索当前搜索状态极小值---------------------------------//
   //alpha 祖先节点得到的当前最小最大值，用于alpha 剪枝
  //beta  祖先节点得到的当前最大最小值，用于beta 剪枝
  //step  还要搜索的步数
 //return 当前搜索子树极小值。
  protected int findMin(int alpha, int beta, int step) {
  	int min = beta;
      if (step == 0) {
          return evaluate();
      }
      int[][] rt = getBests(sbw);
      for (int i = 0; i < rt.length; i++) {
          int x = rt[i][0];
          int y = rt[i][1];
          int type = getType(x, y, sbw);
          if (type == 1)     					  			//玩家成5
              return -100 * ( getMark(1) + step*1000 );
          // 预存当前边界值
          int temp1=x_min,temp2=x_max,temp3=y_min,temp4=y_max;
          isChessOn[x][y] = sbw;
          resetMaxMin(x,y);
          int t = findMax( alpha, min, step - 1 );
          isChessOn[x][y] = 2;
          // 还原预设边界值
          x_min=temp1;
          x_max=temp2;
          y_min=temp3;
          y_max=temp4;
          if (t < min)
          	min = t;
          //alpha 剪枝
          if (min <= alpha) {
              return min;
          }
      }
      return min;
  }


   //-----------------选取局部最优的几个落子点作为下一次扩展的节点---------//
   //bwf 棋色 0：黑棋 1：白棋
   //return 选出来的节点坐标
  private int[][] getBests(int bwf) {

      int i_min=(x_min==0 ? x_min:x_min-1);
      int j_min=(y_min==0 ? y_min:y_min-1);
      int i_max=(x_max==15 ? x_max:x_max+1);
      int j_max=(y_max==15 ? y_max:y_max+1);
      int n = 0;
      int type_1,type_2;
      int[][] rt = new int[(i_max-i_min) * (j_max-j_min)][3];
      for ( int i = i_min; i < i_max; i++) 
      	for (int j = j_min; j < j_max; j++)
      		if (isChessOn[i][j] == 2) {
                  type_1 = getType(i, j, bwf);
                  type_2 = getType(i, j, 1 - bwf);
                  if(able_flag && bwf==0 && (type_1 == 20 || type_1 == 21 || type_1 == 22)) // 禁手棋位置,不记录
                  	continue;
                  rt[n][0] = i;
                  rt[n][1] = j;
                  rt[n][2] = getMark(type_1) + getMark(type_2);
                  n++;
      }
      // 对二维数组排序
      Arrays.sort(rt, new ArrComparator());
      int size = weight > n? n:weight;
      int[][] bests = new int[size][3];
      System.arraycopy(rt, 0, bests, 0, size);
      return bests;
  }

   //----------------------------计算指定方位上的棋型-------------------//
   // x,y 方向线基准一点。
   //ex,ey 指定方向步进向量。
   // k 棋子颜色，0：黑色，1：白色
   // 该方向上的棋子数目 以及 活度
  private int[] count(int x, int y, int ex, int ey, int bwf) {
  	// 该方向没意义,返回0
      if( !makesense(x, y, ex, ey, bwf))
          return new int[] {0, 1};
      
      // 正方向 以及 反方向棋子个数
  	int rt_1 = 1,rt_2 = 1;
  	// 总棋子个数
  	int rt = 1;
  	// 正方向 以及 反方向连子的活度
      int ok_1 = 0,ok_2 =0;
      // 总活度
      int ok = 0;
      // 连子中间有无空格
      boolean flag_mid1 =false,flag_mid2 = false;
      // 连子中间空格的位置
      int flag_i1 = 1,flag_i2 = 1;
      
      if (isChessOn[x][y] != 2) {
          throw new IllegalArgumentException("position x,y must be empty!..");
      }
      int i;
      // 往正方向搜索
      for (i = 1; x + i * ex < 15 && x + i * ex >= 0 && y + i * ey < 15 && y + i * ey >= 0; i++) {
          if (isChessOn[x + i * ex][y + i * ey] == bwf)
              rt_1++;
      	// 位置为空,若中空标志为false,则记为中空并继续搜索  否则,break
          else if(isChessOn[x + i * ex][y + i * ey] == 2) {
          		if(!flag_mid1) {
          			flag_mid1 = true;
          			flag_i1 = i;
          		}
          		else 
          			break;
          	}
          // 位置为对方棋子
          else    
          	break;
      }
      // 计算正方向活度,,
      // 最后一个位置不超过边界
      if (x + i * ex < 15 && x + i * ex >= 0 && y + i * ey < 15 && y + i * ey >= 0) {
      	// 最后一个位置为空位 +1活
      	if( isChessOn[x + i * ex][y + i * ey] == 2) {
      		ok_1++;
      		// 若是在尾部检测到连续的空格而退出搜索,则不算有中空
              if(rt_1 == flag_i1)
      			flag_mid1 = false;
              // 若中空的位置在4以下 且 棋子数>=4,则这一边的4非活
              if(flag_mid1 && rt_1 > 3 && flag_i1 < 4) {
              	ok_1--;
              }
      	}
      	// 最后一个位置不是空格,且搜索了2步以上,若前一个是空格,  则不算中空,且为活的边
      	else if( isChessOn[x + i * ex][y + i * ey] != bwf && i >= 2) 
          	if(isChessOn[x + (i-1) * ex][y + (i-1) * ey] == 2) {
          		ok_1++;
          		flag_mid1 = false;
          	}
      }
      // 最后一个位置是边界  搜索了2步以上,且前一个是空格,  则不算中空,且为活的边
      else if(i >= 2 && isChessOn[x + (i-1) * ex][y + (i-1) * ey] == 2) {
      	ok_1++;
      	flag_mid1 = false;
      }
      
      // 往反方向搜索        
      for (i = 1; x - i * ex >= 0 && x - i * ex < 15 && y - i * ey >= 0 && y - i * ey < 15; i++) {
          if (isChessOn[x - i * ex][y - i * ey] == bwf)
              rt_2++;
          else if(isChessOn[x - i * ex][y - i * ey] == 2) {
          		if(!flag_mid2) {
          			flag_mid2 = true;
          			flag_i2 = i;
          		}
          		else
          			break;
          	}
          else
              break;
      }
      // 计算反方向活度
      if (x - i * ex < 15 && x - i * ex >= 0 && y - i * ey < 15 && y - i * ey >= 0) {
      	if( isChessOn[x - i * ex][y - i * ey] == 2) {
      		ok_2++;
      		if(rt_2 == flag_i2)
      			flag_mid2 = false;
      	    if(flag_mid2 && rt_2 > 3 && flag_i2 < 4) {
              	ok_2--;
              }
      	}
      	else if( isChessOn[x - i * ex][y - i * ey] != bwf && i >= 2 ) 
      		if(isChessOn[x - (i-1) * ex][y - (i-1) * ey] == 2) {
      			ok_2++;
      			flag_mid2 = false;
      		}
      }
      else if(i >= 2 && isChessOn[x - (i-1) * ex][y - (i-1) * ey] == 2) {
      	ok_2++;
  		flag_mid2 = false;
      }
      
      //------------------分析棋子类型
      // 两边都没中空,直接合成
      if( !flag_mid1 && !flag_mid2 ) {
      	rt = rt_1 + rt_2 - 1;
      	ok = ok_1 + ok_2;
      	return new int[] {rt, ok};
      }
      // 两边都有中空
      else if( flag_mid1 && flag_mid2 ){
      	int temp = flag_i1 + flag_i2 - 1;
      	// 判断中间的纯连子数,在5以上,直接返回;  为4,返回活4;  
      	if(temp >= 5)
      		return new int[] {temp, 2};
      	if(temp == 4) 
      		return new int[] {temp, 2};
      	// 先看有没死4,再看有没活3,剩下只能是死3
      	if(rt_1 + flag_i2 - 1 >= 4 || rt_2 + flag_i1 - 1 >= 4) 
      		return new int[] {4, 1};
      	if(rt_1+flag_i2-1 == 3 && ok_1 > 0 || rt_2+flag_i1-1 == 3 && ok_2 > 0)
      		return new int[] {3, 2};
      	return new int[] {3, 1};
      }
      // 有一边有中空
      else {
      	// 总棋子数少于5,直接合成
      	if( rt_1 + rt_2 - 1 < 5 )
      		return new int[] {rt_1 + rt_2 - 1, ok_1 + ok_2};
      	// 多于5,先找成5,再找活4,剩下的只能是死4
      	else {
      		if(flag_mid1 && rt_2 + flag_i1 - 1 >= 5) 
      			return new int[] {rt_2 + flag_i1 - 1, ok_2 + 1};
      		if(flag_mid2 && rt_1 + flag_i2 - 1 >= 5) 
      			return new int[] {rt_1 + flag_i2 - 1, ok_1 + 1};
      		
      		if(flag_mid1 && (rt_2 + flag_i1 - 1 == 4 && ok_2 == 1 || flag_i1 == 4) )
      			return new int[] {4, 2};
      		if(flag_mid2 && (rt_1 + flag_i2 - 1 == 4 && ok_1 == 1 || flag_i2 == 4) )
      			return new int[] {4, 2};
      		
      		return new int[] {4, 1};
      	}
      }
  }

   //----------------------------判断指定方向下棋是否有意义,即最大可能的棋子数是否 >=5-------------------------------//
   // x,y 评估的基准点
   // ex,ey 方向向量
   // k 棋色
   // true:有意义 false:没意义
  private Boolean makesense(int x, int y, int ex, int ey, int bwf) {

      int rt = 1;
      for (int i = 1; x + i * ex < 15 && x + i * ex >= 0 && y + i * ey < 15 && y + i * ey >= 0 && rt < 5; i++)
          if (isChessOn[x + i * ex][y + i * ey] != 1 - bwf)
              rt++;
          else
              break;

      for (int i = 1; x - i * ex >= 0 && x - i * ex < 15 && y - i * ey >= 0 && y - i * ey < 15 && rt < 5; i++)
          if (isChessOn[x - i * ex][y - i * ey] != 1 - bwf)
              rt++;
          else
              break;
      return (rt >= 5);
  }

   //------------------------------------ 棋型判别-------------------------------------//
   // x,y 落子位置
   // bwf 棋色  0：黑子，1：白子
   // 对应的棋型： 棋型代码对应如下：
   //             1：成5
   //             2：成活4或者是双死4或者是死4活3
   //             3：成双活3
   //             4：成死3活3
   //             5：成死4
   //             6：单活3
   //             7：成双活2
  //             8：成死3
   //            9：成死2活2
   //            10：成活2
   //             11：成死2
   //             12: 其他
   //             20: 长连禁手
   //             21: 双四禁手
   //            22: 双活三禁手

  protected int getType(int x, int y, int bwf) {
  	if (isChessOn[x][y] != 2)
          return -1;
  	int[][] types = new int[4][2];
  	types[0] = count(x, y, 0, 1, bwf);   // 竖直
      types[1] = count(x, y, 1, 0, bwf);   // 横向
      types[2] = count(x, y, -1, 1, bwf);  // 斜上
      types[3] = count(x, y, 1, 1, bwf);   // 斜下
      // 各种棋型的方向的数目
      int longfive = 0;
      int five_OR_more = 0;
      int four_died = 0, four_live = 0;
      int three_died = 0, three_live = 0;
      int two_died  = 0, two_live = 0;
      // 各方向上棋型的判别
      for (int k = 0; k < 4; k++) {
      	if (types[k][0] > 5) {  
      		longfive++;              // 长连
      		five_OR_more++;
      	}
      	else if (types[k][0] == 5)
      		five_OR_more++;          // 成5
          else if (types[k][0] == 4 && types[k][1] == 2)
          	four_live++;             // 活4
          else if (types[k][0] == 4 && types[k][1] != 2)
          	four_died++;             // 死4
          else if (types[k][0] == 3 && types[k][1] == 2)
          	three_live ++;           // 活3
          else if (types[k][0] == 3 && types[k][1] != 2)
          	three_died++;            // 死3
          else if (types[k][0] == 2 && types[k][1] == 2)
          	two_live++;              // 活2
          else if (types[k][0] == 2 && types[k][1] != 2)
          	two_died++;              // 死2
          else
              ;
      }
      // 总棋型的判别
      if(bwf == 0 && able_flag) {  		// 黑棋且选择有禁手
      	if (longfive != 0)        		// 长连禁手
      		return 20;
      	if (four_live + four_died >=2)  // 双4禁手
      		return 21;
      	if (three_live  >=2)        	// 双活三禁手
      		return 22;
      }
      if (five_OR_more != 0)
          return 1;   // 成5
      if (four_live != 0 || four_died >= 2 || four_died != 0 && three_live  != 0)
          return 2;   // 成活4或者是双死4或者是死4活3
      if (three_live  >= 2)
          return 3;   // 成双活3
      if (three_died != 0 && three_live  != 0)
          return 4;   // 成死3活3
      if (four_died != 0)
          return 5;   // 成死4
      if (three_live  != 0)
          return 6;   // 单活3
      if (two_live >= 2)
          return 7;   // 成双活2
      if (three_died != 0)
          return 8;   // 成死3
      if (two_live != 0 && two_died != 0)
          return 9;   // 成死2活2
      if (two_live != 0)
          return 10;  // 成活2
      if (two_died != 0)
          return 11;  // 成死2
      return 12;
  }

   //--------------------------对当前棋面进行打分------------------------------------------------------------//

  protected int evaluate() {
  	int rt = 0, mt_c = 1, mt_m = 1;
  	if(bw == sbw)
  		mt_m = 2;
  	else
  		mt_c = 2;
  	int i_min=(x_min==0 ? x_min:x_min-1);
      int j_min=(y_min==0 ? y_min:y_min-1);
      int i_max=(x_max==15 ? x_max:x_max+1);
      int j_max=(y_max==15 ? y_max:y_max+1);
      for (int i = i_min; i < i_max; i++)
          for (int j = j_min; j < j_max; j++)
              if (isChessOn[i][j] == 2) {
              	// 电脑棋面分数
                  int type = getType(i, j, 1 - sbw );
                  if(type == 1)      // 棋型1,棋型2以及棋型3,加权.  防止"4个双活3"的局分大于"1个双四"之类的错误出现
                  	rt += 30 * mt_c * getMark(type);
                  else if(type == 2)					
                  	rt += 10 * mt_c * getMark(type);
                  else if(type == 3)
                  	rt += 3 * mt_c * getMark(type);
                  else
                  	rt += mt_c * getMark(type);
                  // 玩家棋面分数
                  type = getType(i, j, sbw );
                  if(type == 1)
                  	rt -= 30 * mt_m * getMark(type);
                  else if(type == 2)					
                  	rt -= 10 * mt_m * getMark(type);
                  else if(type == 3)
                  	rt -= 3 * mt_m * getMark(type);
                  else
                  	rt -= mt_m * getMark(type);
              }
      return rt;
  }

   //--------------------------------下棋后,更新信息-----------------------------//
  void update(int x,int y) {
  	isChessOn[x][y] = bw;
      bw = 1 - bw;
      pre[chess_num][0] = x;
      pre[chess_num][1] = y;
      chess_num++;
  }
  
   //-------------------------------------- 下棋后,重设边界值------------------------------//
   // x 当前下棋位置的x坐标
   // y 当前下棋位置的y坐标

  public void resetMaxMin(int x,int y){
		if(x-1>=0)
      	x_min = (x_min<x-1 ? x_min:x-1);
      if(x+1<=15)
      	x_max = (x_max>x+1 ? x_max:x+1);
      if(y-1>=0)
      	y_min = (y_min<y-1 ? y_min:y-1);
      if(y+1<=15)
      	y_max = (y_max>y+1 ? y_max:y+1);
  
  }
  

   //------------------------------------------对分数相同的落子点，随机选取-------------------//
   //   kt 随机因子 值越小，被选取的概率越大
   //  return 是否选择该位置

  private boolean randomTest(int kt) {
      Random rm = new Random();
      return rm.nextInt() % kt == 0;
  }


   //------------------------------------- 不同棋型对应分数---------------------------------
   // k 棋型代号
   //return 对应分数
  private int getMark(int k) {
      switch (k) {
      case 1:                   
          return 100000;
      case 2:                   
          return 30000;
      case 3:
          return 5000;
      case 4:
          return 1000;
      case 5:
          return 500;
      case 6:
          return 200;
      case 7:
          return 100;
      case 8:
          return 50;
      case 9:
          return 10;
      case 10:
          return 5;
      case 11:
          return 3;
      case 12:
       	  return 2;
      default:                     //禁手棋型
          return 0;
      }
  }

   //--------------------------------------- 判断是否已分出胜负---------------------------------------------
   // x 落子点x坐标    y 落子点y坐标
   // bwf 棋色 0:黑色 1：白色
   // return true:分出胜负 false:未分出胜负

  public boolean haveWin(int x, int y, int bwf) {
      boolean flag = false;
      if (count(x, y, 1, 0, bwf)[0] >= 5)
          flag = true;
      if (!flag && count(x, y, 0, 1, bwf)[0] >= 5)
          flag = true;
      if (!flag && count(x, y, 1, 0, bwf)[0] >= 5)
          flag = true;
      if (!flag && count(x, y, 1, -1, bwf)[0] >= 5)
          flag = true;
      if (!flag && count(x, y, 1, 1, bwf)[0] >= 5)
          flag = true;
      // 测试用,激活此行代码,不会有输赢..   flag = false;
      return flag;
  }

  public void wined(int bw) {
	  boolean hh=getHumanhuman();
	  if(!hh){           //不是人人对弈
  	       win = true;
           win_bw = bw;
           String str = (bw == sbw ? "恭喜！你赢了！" : "电脑赢了,你还要继续努力啊！");
           if(bw==sbw)
    	        winVoice();
           else
    	        lostVoice();
           JOptionPane.showMessageDialog(null,str);
	  }
	  else{             //人人对弈
		  win = true;
          win_bw = bw;
          String str = (bw == BLACK_ONE ? "恭喜！黑棋获胜！" : "恭喜！白棋获胜！");
   	      winVoice();
          JOptionPane.showMessageDialog(null,str);
	  }
  }
public void setHumanhuman(boolean humanhuman) {
	this.humanhuman = humanhuman;
}
public boolean getHumanhuman() {
	return humanhuman;
}
}
import javax.swing.*;

import java.awt.*;
import java.awt.event.*;
import java.net.URL;
@SuppressWarnings("serial")

public class ChessMap extends JFrame {
	private ImageIcon map;				//棋盘背景位图
	private ImageIcon blackchess;		//黑子位图
	private ImageIcon whitechess;		//白子位图
	private ChessPanel cp;				//棋盘
	private JPanel east;
	private JPanel west;
	private static final int FINAL_WIDTH = 450;
	private static final int FINAL_HEIGHT = 500;
	//以下为下拉菜单
	private JMenuBar menubar;			
	private JMenu[] menu={new JMenu("开始"),new JMenu("设置"),new JMenu("帮助")};
	private JMenuItem[] menuitem1={new JMenuItem("重新开始"),new JMenuItem("悔棋"),new JMenuItem("退出")};
	private JMenuItem[] menuitem2={new JMenuItem("禁手选择"),new JMenuItem("人机博弈"),new JMenuItem("人人对弈")};
	private JMenuItem[] menuitem3={new JMenuItem("规则"),new JMenuItem("关于")};
	private boolean haveai=true;		//人与人下还是人与电脑下，true与电脑下
	Mouseclicked mouseclicked=new Mouseclicked();
	MouseMoved mousemoved=new MouseMoved();
	Menuitemclicked menuclicked=new Menuitemclicked();
	
	//构造函数
	public ChessMap(){
		//改变系统默认字体
		Font font = new Font("Dialog", Font.PLAIN, 12);
		java.util.Enumeration keys = UIManager.getDefaults().keys();
		while (keys.hasMoreElements()) {
			Object key = keys.nextElement();
			Object value = UIManager.get(key);
			if (value instanceof javax.swing.plaf.FontUIResource) {
				UIManager.put(key, font);
			}
		}
		setTitle("五子棋游戏 ");
		setSize(FINAL_WIDTH,FINAL_HEIGHT);
		setResizable(false);
		init();
		setLocation(Toolkit.getDefaultToolkit().getScreenSize().width / 2
				- FINAL_WIDTH / 2, Toolkit.getDefaultToolkit()
				.getScreenSize().height
				/ 2 - FINAL_HEIGHT / 2);
		setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
		cp.reset();	
		setVisible(true);
	}
	
	//初始化与默认值
 	public void init() 
 	{
 		
		map=new ImageIcon(getClass().getResource("bg.jpg"));
		blackchess=new ImageIcon(getClass().getResource("blackchess.gif"));
		whitechess=new ImageIcon(getClass().getResource("whitechess.gif"));
		cp=new ChessPanel(map,blackchess,whitechess);
		menubar=new JMenuBar();
		menuitem1[0].setActionCommand("Restart");
		menuitem1[1].setActionCommand("Rollback");
		menuitem1[2].setActionCommand("Exit");
		menuitem2[0].setActionCommand("Forbid");
		menuitem2[1].setActionCommand("Robot");
		menuitem2[2].setActionCommand("Human");
		menuitem3[0].setActionCommand("Rule");
		menuitem3[1].setActionCommand("About");
		for(int i=0;i<3;i++)
			menu[0].add(menuitem1[i]);
		for(int i=0;i<3;i++)
			menu[1].add(menuitem2[i]);
		for(int i=0;i<2;i++)
			menu[2].add(menuitem3[i]);
		for(int i=0;i<3;i++)
			menubar.add(menu[i]);
		Container p = getContentPane();
		setJMenuBar(menubar);
		east = new JPanel();
		west = new JPanel();
		p.add(east, "East");
		p.add(west, "West");
		p.add(cp, "Center");
		cp.addMouseListener(mouseclicked);
		cp.addMouseMotionListener(mousemoved);
		menuitem1[0].addActionListener(menuclicked);
		menuitem1[1].addActionListener(menuclicked);
		menuitem1[2].addActionListener(menuclicked);
		menuitem2[0].addActionListener(menuclicked);
		menuitem2[1].addActionListener(menuclicked);
		menuitem2[2].addActionListener(menuclicked);
		menuitem3[0].addActionListener(menuclicked);
		menuitem3[1].addActionListener(menuclicked);
	
	}
	class Mouseclicked extends MouseAdapter		//判断鼠标左击并通知棋盘和电脑
	{
		public void mouseClicked(MouseEvent e)
		{
		  if(cp.win==false){
			    if(haveai){           //和电脑博弈
    		              Point p1=new Point();
    		              p1=cp.getPoint(e.getX(),e.getY());
    		              int x=p1.x;
    		              int y=p1.y;
                          // 如果该位置已经放置棋子
    		              System.out.println("x="+x+",y="+y);
                          if (cp.isChessOn[x][y] != 2)
                                     return;
                          // 玩家为黑棋,考虑禁手
                          if( cp.able_flag && cp.bw == 0) {
             	                 int type = cp.getType(x,y,cp.bw);
             	                 String str = null;
             	                 switch(type){
             		             case 20: 
             			               str = "黑长连禁手!请选择其它位置下棋!";
             			               break;
             		             case 21:
             			               str = "黑四四禁手!请选择其它位置下棋!";
             			               break;
             		             case 22: 
             			               str = "黑三三禁手!请选择其它位置下棋!";
             			               break;
             		             default : break;
             	                 }
             	                 if(str != null) {
             		                    JOptionPane.showMessageDialog(null,str);
             		                     return;
             	                 }
     		              }
                          boolean flag=cp.haveWin(x, y, cp.bw);
                          cp.update( x, y );
                          cp.putVoice();  //落子声音
                          // 第一步棋,需初始化设置边界值
                         if( cp.chess_num == 1){  
                      	 if(x-1>=0)
                     	          cp.x_min = x-1;
                         if(x-1<=15)
                     	          cp.x_max = x+1;
                         if(y-1>=0)
                     	          cp.y_min = y-1;
                         if(y-1<=15)
                     	          cp.y_max = y+1;
                  }
                 else 
                 	cp.resetMaxMin(x,y);
                 if (flag) {
                     cp.wined(1 - cp.bw);
                     return;
                 }
                 cp.putOne(cp.bw);
			}else{                                        //和人博弈
				Point p1=new Point();
 		        p1=cp.getPoint(e.getX(),e.getY());
 		        int x=p1.x;
 		        int y=p1.y;
                 // 如果该位置已经放置棋子
 		        System.out.println("x="+x+",y="+y);
                 if (cp.isChessOn[x][y] != 2)
                             return;
                 // 玩家为黑棋,考虑禁手
                 if( cp.able_flag && cp.bw == 0) {
          	           int type = cp.getType(x,y,cp.bw);
          	           String str = null;
          	           switch(type){
          		       case 20: 
          			       str = "黑长连禁手!请选择其它位置下棋!";
          			       break;
          		       case 21:
          			       str = "黑四四禁手!请选择其它位置下棋!";
          			       break;
          		       case 22: 
          			       str = "黑三三禁手!请选择其它位置下棋!";
          			       break;
          		       default : break;
          	           }
          	           if(str != null) {
          		               JOptionPane.showMessageDialog(null,str);
          		               return;
          	           }
  		       }
                boolean flag=cp.haveWin(x, y, cp.bw);
                cp.update( x, y );
                cp.putVoice();  //落子声音
                cp.repaint();
              // 第一步棋,需初始化设置边界值
              if( cp.chess_num == 1){  
              	if(x-1>=0)
                  	cp.x_min = x-1;
                  if(x-1<=15)
                  	cp.x_max = x+1;
                  if(y-1>=0)
                  	cp.y_min = y-1;
                  if(y-1<=15)
                  	cp.y_max = y+1;
              }
              else 
              	cp.resetMaxMin(x,y);
              if (flag) {
                  cp.wined(1 - cp.bw);
                  return;
              }
			}
    	} 
		}
	}
	class MouseMoved implements MouseMotionListener		//调试用，获得鼠标位置
	{
		public void mouseMoved(MouseEvent e)
    	{
    		cp.showMousePos(e.getPoint());
    	}
    	public void mouseDragged(MouseEvent e)
    	{}
	}
	class Menuitemclicked implements ActionListener		//菜单消息处理
	{
		public void actionPerformed(ActionEvent e) 
		{
      		JMenuItem target = (JMenuItem)e.getSource();
      		String actionCommand = target.getActionCommand();
      		if(actionCommand.equals("Restart")){ 		//重开一局
        	   cp.reset();	
        	   if(cp.sbw==cp.WHITE_ONE)
        		   cp.update(7, 7); 
        	   //player=cp.BLACK_ONE;
      		}
      		if(actionCommand.equals("Rollback")){ 		//悔棋
      			if(cp.win) {
        			JOptionPane.showMessageDialog(null,"棋局已经结束,不能悔棋!请重新开始新的棋局!");
        			return;
                }
        		// 当前轮到玩家下棋,取消两步  否则,取消一步
        		if(cp.chess_num >= 2 && cp.bw == cp.sbw){
        			cp.isChessOn[cp.pre[cp.chess_num-1][0]][cp.pre[cp.chess_num-1][1]] = 2;
        			cp.isChessOn[cp.pre[cp.chess_num-2][0]][cp.pre[cp.chess_num-2][1]] = 2;
        			cp.chess_num -= 2;
        			cp.repaint();
        		}
        		else if(cp.chess_num >= 1 && cp.bw == 1-cp.sbw){
        			cp.isChessOn[cp.pre[cp.chess_num-1][0]][cp.pre[cp.chess_num-1][1]] = 2;
        			cp.chess_num --;
       				cp.repaint();
       			}
      		}
      		else if(actionCommand.equals("Exit")){ 		//退出
        		System.exit(1);	
      		}
      		else if(actionCommand.equals("Forbid")){     //禁手选择
        		Object[] options = { "无禁手", "有禁手" };
        		int sel = JOptionPane.showOptionDialog(
          				null, "你的选择：", "禁手选择",
          				JOptionPane.DEFAULT_OPTION,
          				JOptionPane.QUESTION_MESSAGE, null,
          				options, options[0]);
          		if(sel==1){
                        cp.able_flag=true;
                        System.out.println("有禁手");
          		}else{
          			    cp.able_flag=false;
                        System.out.println("无禁手");
          		}
          	}
      		else if(actionCommand.equals("Robot")){            //人机博弈
      			haveai=true;
      			Object[] options = { "人类先手", "机器先手" };
        		int sel = JOptionPane.showOptionDialog(
          				null, "你的选择：", "先手选择",
          				JOptionPane.DEFAULT_OPTION,
          				JOptionPane.QUESTION_MESSAGE, null,
          				options, options[0]);
          		if(sel==1){       //机器先手
          			    cp.sbw=cp.WHITE_ONE;
          			    cp.update(7, 7);
          			    System.out.println("机器先手");
         			    
          		}else{             //人先手
          			    //player=cp.BLACK_ONE;
          			    cp.sbw=cp.BLACK_ONE;
          			    System.out.println("人先手");
          		}
      		}
          	else if(actionCommand.equals("Human")){ 		//人人博弈
        		haveai=false;	
        		cp.setHumanhuman(true);
      		}else if(actionCommand.equals("Rule")){          //规则
      			JOptionPane.showConfirmDialog(null,
      			"1、无禁手：" +"\n"+
				"   黑白双方依次落子，任一方先在棋盘上形成连续的五个(含五个以上)棋子的一方为胜。" +"\n"+
				"2、有禁手：（走禁手就输，禁手不能落子）" +"\n"+
				"   鉴于无禁手规则黑棋必胜，人们不断采用一些方法限制黑棋先行的优势，以平衡黑白双方的形式。" +"\n"+
				"   于是针对黑棋的各种禁手逐渐形成。" +"\n"+
				"   禁手主要分为以下几类：" +"\n"+
				"   (1)黑长连禁手：连成六个以上连续相同的棋子。" +"\n"+
				"   (2)黑三三禁手：两个以上的活三。" + "\n"+
				"   (3)黑四四禁手：两个以上的四。" + "\n"+
				"   禁手是针对黑棋而言的，白棋没有任何禁手。" ,"规则",JOptionPane.CLOSED_OPTION,JOptionPane.INFORMATION_MESSAGE);
      		}
      		else if(actionCommand.equals("About")){ 		//版权与帮助
        		JOptionPane.showConfirmDialog(null,"制作人：徐晓萌 郭蕾\n计科高职13-2\n201303014063","关于",JOptionPane.CLOSED_OPTION,JOptionPane.INFORMATION_MESSAGE);	
      		}
    	}

	}
  public static void main(String[] args) {
	    new ChessMap();	
  }
} 


import java.util.Comparator;

/**
 * 排序 Comparator
 */
class ArrComparator implements Comparator<Object> {
    int column = 2;

    int sortOrder = -1; // 递减

    public ArrComparator() {
    }
    public int compare(Object a, Object b) {
        if (a instanceof int[]) {
            return sortOrder * (((int[]) a)[column] - ((int[]) b)[column]);
        }
        throw new IllegalArgumentException("param a,b must int[].");
    }
}

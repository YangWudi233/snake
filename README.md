![](media/image1.png)

**Java程序设计课程设计报告**

> **设计题目 *贪吃蛇 ***
>
> **年级专业 *计算机科学与技术 ***
>
> **学 号 *22330472013 ***
>
> **学生姓名 *杨振豪 ***
>
> **指导教师 *张倬维 ***

2022年12月 1日

**目 录（例）**

1 项目简介 [1](#项目简介)

2 设计流程 [2](#设计流程)

3 程序代码 [3](#_Toc30669)

3.1游戏角色类设计 [3](#_Toc30697)

3.2工具类设计 [3](#coordinate)

4 运行结果 [4](#_Toc27199)

5 调试优化 5

6 设计总结 6

# 1 项目简介

贪吃蛇游戏是一个深受人们喜爱的游戏，一条在密闭的围墙内，在围墙内随机出现一 个食物，通过按键盘上四个光标键控制蛇向上下左右四个方向移动，蛇头撞到食物，则表示食物被蛇吃掉，这时蛇的身体长一节，同时计1分，接着又出现食物，等待被蛇吃掉，如果蛇在移动过程中，身体交叉蛇头撞到自己的身体游戏结束.

GitHub链接：https://github.com/YangWudi233/snake

#  2 设计流程

**2.1绘制窗体对象。**

**2.2 静态UI设计（包括小蛇，食物，游戏区域和标题区域）。**

**2.3使用键盘监听事件和定时器实现小蛇的移动。**

**2.4小蛇与食物碰撞的实现。**

**2.5定义变量存放小蛇长度，遍历数组实现小蛇身体的增加功能。**

**2.6退出条件：当游戏积分到达指定分数，游戏退出。**

**3 程序代码**

**3.1snake类**

public class Snake {  
private Scene GameUI;//母窗体,即游戏主界面  
public Direction direction = Direction.RIGHT;//蛇当前前进的方向,初始化默认向右移动  
private Deque\<Coordinate\> body = new LinkedList\<\>();//用于描述蛇身体节点的数组，保存蛇身体各个节点的坐标  
private ScheduledExecutorService executor;//刷新线程  
private Coordinate food;//食物坐标  

public Snake(Scene GameUI){  
this.GameUI = GameUI;  
Coordinate head = new Coordinate(0, 0);//初始化头部在(0,0)位置  
body.addFirst(head);  
produceFood();  
run();  
}  

public Coordinate randomCoor(){  
int rows = GameUI.height, cols = GameUI.width;  
Random rand = new Random();  
Coordinate res;  
int x = rand.nextInt(cols-1);  
int y = rand.nextInt(rows-1);  

while(true) {  
boolean tag = false;  
for(Coordinate coor : body){  
if(x == coor.y && y == coor.x){  
x = rand.nextInt(cols-1);  
y = rand.nextInt(rows-1);  
tag = true;  
break;  
}  
}  

if(!tag){  
break;  
}  
}  
res = new Coordinate(x, y);  
return res;  
}  

//蛇身体移动  
public void move(){  
Coordinate head, next_coor = new Coordinate(0,0);  
if(direction == Direction.UP){  
head = body.getFirst();//获取头部  
next_coor = new Coordinate(head.x,head.y - 1);//头部向上移动一个单位后的坐标  
} else if(direction == Direction.DOWN){  
head = body.getFirst();//获取头部  
next_coor = new Coordinate(head.x,head.y + 1);//头部向下移动一个单位后的坐标  
} else if(direction == Direction.LEFT){  
head = body.getFirst();//获取头部  
next_coor = new Coordinate(head.x - 1,head.y);//头部向左移动一个单位后的坐标  
} else if(direction == Direction.RIGHT){  
head = body.getFirst();//获取头部  
next_coor = new Coordinate(head.x + 1,head.y);//头部向右移动一个单位后的坐标  
}  

if(checkDeath(next_coor)) {//判断下一步是否死亡  
new Music("music//over.wav").start();  
GameUI.quit = true;  
GameUI.pause = true;  
GameUI.die = true;  
GameUI.repaint();  
} else {  
Coordinate next_node = new Coordinate(next_coor);  
body.addFirst(next_node);//添头  
if(!checkEat(next_coor)) {//没吃到食物就去尾，否则不用去掉，因为添加的头刚好是吃到一个食物后增长的一节  
body.pollLast();//去尾  
}else{  
new Music("music//eat.wav").start();  
GameUI.updateLength(body.size());  
produceFood();  
}  
}  
}  

//判断一个坐标位置是否是蛇死亡的位置  
public boolean checkDeath(Coordinate coor){  
int rows = GameUI.height, cols = GameUI.width;  
return coor.x \< 0 \|\| coor.x \>= cols \|\| coor.y \< 0 \|\| coor.y \>= rows;  
}  

public boolean checkEat(Coordinate coor){  
return food.x == coor.x && food.y == coor.y;  
}  

public Deque\<Coordinate\> getBodyCoors(){  
return body;  
}  

public Coordinate getFoodCoor(){  
return food;  
}  

public void produceFood(){  
food = randomCoor();  
}  

public void show(){  
GameUI.repaint();  
}  

public void quit(){  
executor.shutdownNow();//退出线程  
}  

public void run(){  
executor = Executors.newSingleThreadScheduledExecutor();  
int speed = 500;//用于描述蛇移动速度的变量，其实是作为蛇刷新线程的时间用的  
executor.scheduleAtFixedRate(() -\> {  
if (!GameUI.pause && !GameUI.quit) {  
move();  
show();  
}  
}, 0, speed, TimeUnit.MILLISECONDS);  
}  
}

### 3.1.1body

private Deque\<Coordinate\> body = new LinkedList\<\>();

//用于描述蛇身体节点的数组，保存蛇身体各个节点的坐标

body这个队列保存了我们这条蛇每一个结点的坐标，这个 Coordinate 类比较简单，就是封装了x坐标和y坐标的值，因为无论是贪吃蛇，还是食物，或者是障碍物，其实都只是一个坐标，这个和它长什么样子，如何显示是没关系的，所以我们需要准备好这个后面会经常用到的数据结构。

### 3.1.2Coordinate() 

//坐标数据结构，用于表示蛇身体，食物，障碍物的坐标

//这里的坐标用方块的序号表示，实际显示时再换成屏幕真实坐标(即像素点)

public class Coordinate {

public int x,y;

public Coordinate(int x0,int y0){

x = x0;

y = y0;

}

public Coordinate(Coordinate temp){

x = temp.x;

y = temp.y;

}

}

### 3.1.3run()

public void run(){

executor = Executors.newSingleThreadScheduledExecutor();

int speed = 500;//用于描述蛇移动速度的变量，其实是作为蛇刷新线程的时间用的

executor.scheduleAtFixedRate(() -\> {

if (!GameUI.pause && !GameUI.quit) {

move();

show();

}

}, 0, speed, TimeUnit.MILLISECONDS);

}

贪吃蛇在不对它进行控制的时候它自己也是会朝着之前的方向移动的，通过一个线程进行实现，简单来说，就是每隔500毫秒执行一次内部的move()和show()。

这个线程我之前是通过下面这种方式实现的

public void run(){

while(!quit){

try {

Thread.sleep(500);//每隔500毫秒刷新一次

} catch (InterruptedException e) {

e.printStackTrace();

}

move();

show();

}

}

不建议在循环里面去使用Thread.sleep(500)

### 3.1.4move()

//蛇身体移动  
public void move(){  
Coordinate head, next_coor = new Coordinate(0,0);  
if(direction == Direction.UP){  
head = body.getFirst();//获取头部  
next_coor = new Coordinate(head.x,head.y - 1);//头部向上移动一个单位后的坐标  
} else if(direction == Direction.DOWN){  
head = body.getFirst();//获取头部  
next_coor = new Coordinate(head.x,head.y + 1);//头部向下移动一个单位后的坐标  
} else if(direction == Direction.LEFT){  
head = body.getFirst();//获取头部  
next_coor = new Coordinate(head.x - 1,head.y);//头部向左移动一个单位后的坐标  
} else if(direction == Direction.RIGHT){  
head = body.getFirst();//获取头部  
next_coor = new Coordinate(head.x + 1,head.y);//头部向右移动一个单位后的坐标  
}  

if(checkDeath(next_coor)) {//判断下一步是否死亡  
new Music("music//over.wav").start();  
GameUI.quit = true;  
GameUI.pause = true;  
GameUI.die = true;  
GameUI.repaint();  
} else {  
Coordinate next_node = new Coordinate(next_coor);  
body.addFirst(next_node);//添头  
if(!checkEat(next_coor)) {//没吃到食物就去尾，否则不用去掉，因为添加的头刚好是吃到一个食物后增长的一节  
body.pollLast();//去尾  
}else{  
new Music("music//eat.wav").start();  
GameUI.updateLength(body.size());  
produceFood();  
}  
}  
}

move函数里面的内容就是每次刷新的时候贪吃蛇往前走一步，如果吃到了食物，身体就增加一节，然后还要判断一下是不是撞墙死了。

### 3.1.5show()

public void show(){  
GameUI.repaint();  
}

show函数自然是刷新游戏界面了，这个就跟你是如何显示贪吃蛇有关了，如果你是用绘图的方式显示贪吃蛇，那么你就直接调用主界面的repaint函数就行了，如果是贴图的方式，那么你可能要在界面上增加一张图片或者删除一张图片。

**3.2 Scene类**

package com.lan;  
import javax.swing.\*;  
import java.awt.\*;  
import java.awt.event.KeyEvent;  
import java.awt.event.KeyListener;  
import java.util.Deque;  
import java.util.concurrent.Executors;  
import java.util.concurrent.ScheduledExecutorService;  
import java.util.concurrent.TimeUnit;  

public class Scene extends JFrame{  
private final Font f = new Font("微软雅黑",Font.PLAIN,15);  
private final Font f2 = new Font("微软雅黑",Font.PLAIN,12);  
private JPanel paintPanel;//画板，画线条用的  
private final JLabel label = new JLabel("当前长度：");  
private final JLabel label2 = new JLabel("所花时间：");  
private final JLabel Length = new JLabel("1");  
private final JLabel Time = new JLabel("");  
private Timer timer;  
public boolean pause = false;  
public boolean quit = false;  
public boolean die = false;  
private boolean show_padding = true;  
private boolean show_grid = true;  
public final int pixel_per_unit = 22; //每个网格的像素数目  
public final int pixel_rightBar = 110; //右边信息栏的宽度(像素)  
public final int padding = 5; //内边框宽度  
public final int width = 20;  
public final int height = 20;  
private Snake snake;  

public void restart(){//重新开始游戏  
quit = true;  
Length.setText("1");  
Time.setText("");  

snake.quit();  
snake = null;  
snake = new Snake(this);  

timer.reset();  

die = false;  
quit = false;  
pause = false;  

System.out.println("\nGame restart...");  
}  

public void updateLength(int length){  
Length.setText(""+length);  
}  

//通过方格序号返回其横坐标  
public int getPixel(int i, int padding, int pixels_per_unit) {  
return 1+padding+i\*pixels_per_unit;  
}  

public void initUI(){  
String lookAndFeel = UIManager.getSystemLookAndFeelClassName();  
try {  
UIManager.setLookAndFeel(lookAndFeel);  
} catch (ClassNotFoundException \| InstantiationException \| IllegalAccessException \| UnsupportedLookAndFeelException e1) {  
e1.printStackTrace();  
}  

Image img = Toolkit.getDefaultToolkit().getImage("image//title.png");//窗口图标  
setIconImage(img);  
setTitle("Snake");  
setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);  
setSize(width\*pixel_per_unit+pixel_rightBar, height \* pixel_per_unit + 75);  
setResizable(false);  
setLayout(null);  
setLocationRelativeTo(null);  

paintPanel = new JPanel(){  
//绘制界面的函数  
public void paint(Graphics g1){  
super.paint(g1);  
Graphics2D g = (Graphics2D) g1;  
g.setRenderingHint(RenderingHints.KEY_ANTIALIASING,RenderingHints.VALUE_ANTIALIAS_ON);  
g.setRenderingHint(RenderingHints.KEY_STROKE_CONTROL,RenderingHints.VALUE_STROKE_NORMALIZE);  

//边框线  
if(show_padding){  
g.setPaint(new GradientPaint(115,135,Color.CYAN,230,135,Color.MAGENTA,true));  
g.setStroke( new BasicStroke(4,BasicStroke.CAP_BUTT,BasicStroke.JOIN_BEVEL));  
g.drawRect(2, 2, padding\*2-4+width\*pixel_per_unit, height\*pixel_per_unit+6);  
}  

//网格线  
if(show_grid) {  
for(int i = 0; i \<= width; i++) {  
g.setStroke( new BasicStroke(1f, BasicStroke.CAP_BUTT, BasicStroke.JOIN_ROUND,  
3.5f, new float\[\] { 15, 10, }, 0f));//虚线  
g.setColor(Color.black);  
g.drawLine(padding+i\*pixel_per_unit, padding,  
padding+i\*pixel_per_unit,padding+height\*pixel_per_unit);//画竖线  
}  

for(int i = 0;i \<= height; i++){  
g.drawLine(padding,padding+i\*pixel_per_unit,  
padding+width\*pixel_per_unit,padding+i\*22);//画横线  
}  
}  

//食物  
Coordinate food = snake.getFoodCoor();  
g.setColor(Color.green);  
g.fillOval(getPixel(food.x, padding, pixel_per_unit),  
getPixel(food.y, padding, pixel_per_unit), 20, 20);  

//头部  
Deque\<Coordinate\> body = snake.getBodyCoors();  
Coordinate head = body.getFirst();  
g.setColor(Color.red);  
g.fillRoundRect(getPixel(head.x, padding, pixel_per_unit),  
getPixel(head.y, padding, pixel_per_unit), 20, 20, 10, 10);  

//身体  
g.setPaint(new GradientPaint(115,135,Color.CYAN,230,135,Color.MAGENTA,true));  
for (Coordinate coor : body){  
if(head.x == coor.x && head.y == coor.y) continue;  
g.fillRoundRect(getPixel(coor.x, padding, pixel_per_unit),  
getPixel(coor.y, padding, pixel_per_unit), 20, 20, 10, 10);  
}  

//显示死亡信息  
if(die) {  
g.setFont(new Font("微软雅黑",Font.BOLD \| Font.ITALIC,30));  
g.setColor(Color.BLACK);  
g.setStroke( new BasicStroke(10,BasicStroke.CAP_BUTT,BasicStroke.JOIN_BEVEL));  

int x = this.getWidth()/2, y = this.getHeight()/2;  
g.drawString("Sorry, you die", x-350, y-50);  
g.drawString("Press ESC to restart", x-350, y+50);  
}  
}  
};  
paintPanel.setOpaque(false);  
paintPanel.setBounds(0, 0, 900, 480);  
add(paintPanel);  

int info_x = padding\*3 + width\*pixel_per_unit;  
add(label);label.setBounds(info_x, 10, 80, 20);label.setFont(f);label.setForeground(Color.black);  
add(Length);Length.setBounds(info_x, 35, 80, 20);Length.setFont(f);Length.setForeground(Color.black);  
add(label2);label2.setBounds(info_x, 70, 80, 20);label2.setFont(f);label2.setForeground(Color.black);  
add(Time);Time.setBounds(info_x, 95, 80, 20);Time.setFont(f);Time.setForeground(Color.black);  

//菜单栏  
JMenuBar bar = new JMenuBar();bar.setBackground(Color.white);setJMenuBar(bar);  
JMenu Settings = new JMenu("设置");Settings.setFont(f);bar.add(Settings);  
JMenu Help = new JMenu("帮助");Help.setFont(f);bar.add(Help);  
JMenu About = new JMenu("关于");About.setFont(f);bar.add(About);  
JMenuItem remove_net= new JMenuItem("移除网格");remove_net.setFont(f2);Settings.add(remove_net);  
JMenuItem remove_padding= new JMenuItem("移除边框");remove_padding.setFont(f2);Settings.add(remove_padding);  
JMenuItem help = new JMenuItem("Guide...");help.setFont(f2);Help.add(help);  
JMenuItem about = new JMenuItem("About...");about.setFont(f2);About.add(about);  

//监听器  
this.addKeyListener(new MyKeyListener());  
remove_net.addActionListener(e -\> {  
if(!show_grid) {  
show_grid = true;  
remove_net.setText("移除网格");  
} else {  
show_grid = false;  
remove_net.setText("显示网格");  
}  
paintPanel.repaint();  
});  
remove_padding.addActionListener(e -\> {  
if(!show_padding) {  
show_padding = true;  
remove_padding.setText("移除边框");  
} else {  
show_padding = false;  
remove_padding.setText("显示边框");  
}  
paintPanel.repaint();  
});  
about.addActionListener(e -\> new About());  
help.addActionListener(e -\> new Help());  
}  

public void run(){  
snake = new Snake(this);  
setFocusable(true);  
setVisible(true);  
timer = new Timer();  
}  

public static void main(String\[\] args) {  
System.out.println("Application starting...");  
Scene game = new Scene(); //初始化游戏场景  
game.initUI(); //初始化游戏界面  
game.run(); //开始游戏  
System.out.println("Game start...");  
}  

private class Timer{  
//计时器类,负责计时，调用方法，new Timer(); 然后主界面就开始显示计时  
private int hour = 0;  
private int min = 0;  
private int sec = 0;  

public Timer(){  
this.run();  
}  

public void run() {  
ScheduledExecutorService executor = Executors.newSingleThreadScheduledExecutor();  
executor.scheduleAtFixedRate(() -\> {  
if (!quit && !pause) {  
sec +=1 ;  
if(sec \>= 60){  
sec = 0;  
min +=1 ;  
}  
if(min\>=60){  
min=0;  
hour+=1;  
}  
showTime();  
}  
}, 0, 1000, TimeUnit.MILLISECONDS);  
}  

public void reset() {  
hour = 0;  
min = 0;  
sec = 0;  
}  

private void showTime(){  
String strTime;  
if(hour \< 10) strTime = "0"+hour+":";  
else strTime = ""+hour+":";  

if(min \< 10) strTime = strTime+"0"+min+":";  
else strTime =strTime+ ""+min+":";  

if(sec \< 10) strTime = strTime+"0"+sec;  
else strTime = strTime+""+sec;  

//在窗体上设置显示时间  
Time.setText(strTime);  
}  
}  

private class MyKeyListener implements KeyListener{  
@Override  
public void keyPressed(KeyEvent e) {  
int key = e.getKeyCode();  
Direction direction = snake.direction;  

if(key == KeyEvent.VK_RIGHT \|\| key == KeyEvent.VK_D) { //向右  
if(!quit && direction != Direction.LEFT) {  
snake.direction = Direction.RIGHT;  
}  
} else if(key == KeyEvent.VK_LEFT \|\| key == KeyEvent.VK_A) { //向左  
if(!quit && direction != Direction.RIGHT) {  
snake.direction = Direction.LEFT;  
}  
} else if(key == KeyEvent.VK_UP \|\| key == KeyEvent.VK_W) { //向上  
if(!quit && direction != Direction.DOWN) {  
snake.direction = Direction.UP;  
}  
} else if(key == KeyEvent.VK_DOWN \|\| key == KeyEvent.VK_S) { //向下  
if(!quit && direction != Direction.UP) {  
snake.direction = Direction.DOWN;  
}  
} else if(key == KeyEvent.VK_ESCAPE) { //重新开始  
restart();  
} else if(key == KeyEvent.VK_SPACE) {  
if(!pause) {//暂停  
pause = true;  
System.out.println("暂停...");  
} else {//开始  
pause = false;  
System.out.println("开始...");  
}  
}  
}  

@Override  
public void keyReleased(KeyEvent e) {  
// TODO Auto-generated method stub  
}  

@Override  
public void keyTyped(KeyEvent e) {  
// TODO Auto-generated method stub  
}  
}  
}

**4 运行结果**

**4.1贪吃蛇运动功能**

> ![](media/image2.png)

图1 开始图

![](media/image3.png)

图1 结束图

# ** ****5调试优化**

## 5.1问题&调试

开始的时候用的是repaint()调用贴图，但是无法显示图片。百度后没有找到原因，后来选择改成绘图的方式。

## 5.2版本规划

首先就是前面提到的改用贴图的方式显示蛇的本体，其次增加可玩性：例如

1.  玩家蛇可以发射子弹来击毁前面的障碍物，通过吃到特定的食物来获取子弹。

2.  长按方向键可以加速

3.  障碍物每隔一定时间会自动移动和变化形状

4.  食物每隔一段时间会自动刷新

# ** 6 设计总结**

Java是纯面向对象编程（OOP）的一门语言，所以程序中所有的功能都必须封装到某个类中去。

那么我们就需要去思考我们这个游戏需要用到哪些类。事实上，这个过程并不简单，因为我们要去思考游戏中哪些实体会对应一个类，而这个类又需要封装好哪些功能，不同的类之间有哪些通信需求（数据交换），我们怎么让它们能够简单地访问到自己所需要的数据。这些都是值得自己去思考的。

事实上，这个问题我一开始（2017年7月份）思考的也不是很清楚，有些压根就没考虑到，所以随着自己的代码越来越多，各种不合理的问题就会逐渐暴露出来，比如：抽象层次不合理，接口设计不合理。

大家肯定能想到的是，我们的贪吃蛇肯定要是一个单独的类。它内部封装好了一些属于它的数据和方法。同时，它也需要暴露一些接口给外界调用。

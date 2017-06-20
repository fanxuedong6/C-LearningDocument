# JAVA
```
import java.util.*;
import java.io.*;

public class JavaReview {

//Input Ways inputTest()

    //The Safest way to input
    public static void inputScanner() {
        Scanner scan = new Scanner(System.in);
        System.out.println("input name:");
        String name = scan.next();
        System.out.println("input age:");
        int age = scan.nextInt();
        System.out.println("name:" + name + "age" + age);
	}

    //BufferedReader throws out IOExcption
    public static void inputBufferReader() {
        try {
            BufferedReader inputStream = new BufferedReader(new InputStreamReader(System.in));
            System.out.println("input name:");
            String name = inputStream.readLine();
            System.out.println("input number:");
            int age = Integer.valueOf(inputStream.readLine().trim()).intValue();
            System.out.println("name:" + name + "age" + age);
        } catch(Exception e) {
            e.printStackTrace();
        }
    }
    public static void inputTest() {
        inputScanner();
        inputBufferReader();
    }

//Polymorphism shapeTest()

    public static abstract class Shape {
        public abstract double area();
    }
    public static class Triangle extends Shape {
        public double a;
        public double b;
        public double c;
        @Override
        public double area()
        {
            double p = (a + b + c) / 2;
            return Math.sqrt(p * (p - a) * (p - b) * (p - c));
        }
    }
    public static class Circle extends Shape {
        public double r;
        @Override
        public double area()
        {
            return Math.PI * r * r;
        }
    }
    public static void shapeTest() throws IOException {
        Circle c = new Circle();
        c.r = 4;
        Triangle t = new Triangle();
        t.a = 5;
        t.b = 2;
        t.c = 3;
        Shape[] shapes = new Shape[]{c, t};
        for(Shape s : shapes) {
           System.out.println(s.area());
        }
    }

//Interface tuxingTest()

    public interface TuXing {
        double zhouChang(); //perimeter
        double mianJi();
    }
    public static class TriangleInter implements TuXing {
        public double a;
        public double b;
        public double c;
        public double p;

        @Override
        public double zhouChang() {
            return a + b + c;
        }

        @Override
        public double mianJi() {
            return Math.sqrt(p * (p - a) * (p - b) * (p - c));
        }
    }
    public static class CircleInter implements TuXing {
        public double r;
        
        @Override
        public double zhouChang() {
            return 2 * Math.PI * r;
        }

        @Override
        public double mianJi() {
            return Math.PI * r * r;
	    }
    }
    public static void tuxingTest() {
        CircleInter c = new CircleInter();
        c.r = 4;
        TriangleInter t = new TriangleInter();
        t.a = 5;
        t.b = 2;
        t.c = 3;
        t.p = 10;
        TuXing[] tuXings = new TuXing[]{c, t};
        for(TuXing s : tuXings) {
            System.out.println("Area of " + s.getClass().getName() + ": " + s.mianJi());
            System.out.println("Perimeter of " + s.getClass().getName() + ": " + s.zhouChang());
        }
    }

//Math mathTest()

    public static void mathTest() {
	System.out.println("Random number 0 <= x < 100: " + (int)((Math.random() * 100.0)));
	System.out.println("E = " + Math.E);
	System.out.println("60˚ = " + (60.0 / 180.0) + "π = " + (60.0 / 180.0 * Math.PI));
	System.out.println("ln(E) = " + Math.log(Math.E));
	System.out.println("2^4 = " + Math.pow(2, 4));
    }

//String Process stringProcess()

    public static void stringProcess() {
    //substring
        String[] urls = new String[] {
		    "http://www.blcu.edu.cn/",
		    "http://news.blcu.edu.cn/xyxw/421321.shtml",
		    "http://www.nasa.gov/",
		    "http://www.esa.int/ESA",
		    "http://www.nasa.gov/mission_pages/station/main/index.html",
        };	
        for(int i=0; i<urls.length; i++) {
            String u = urls[i];
            u = u.substring(u.indexOf('.') + 1);
            u = u.substring(0, u.indexOf('.'));
            u = u.toUpperCase();
            System.out.println(urls[i] + "; Organization: " + u);
        }

	//charAt, setCharAt
        StringBuffer password = new StringBuffer("234567");
        StringBuffer newPassword = new StringBuffer("");
        for(int i=0; i<password.length(); i++) {
            newPassword.append((Integer.parseInt("" + password.charAt(i)) + 6) % 10);
        }
        char t = newPassword.charAt(1);
        newPassword.setCharAt(1,  newPassword.charAt(4));
        newPassword.setCharAt(4,  t);
        char u = newPassword.charAt(0);
        newPassword.setCharAt(0,  newPassword.charAt(5));
        newPassword.setCharAt(5,  u);
            
        System.out.println("Old password: " + password);
        System.out.println("New password: " + newPassword);

    //"==" will compare the adress, to compare the value, use string.equals(string)
        String StringA = "BeijingTianjinChenjinjinjinyouwei";
        String StringB = "jin";
        int count = 0;
        for(int i=0; i<StringA.length() - StringB.length() + 1; i++) {
            System.out.println(StringA.substring(i, i + 3));
            if(StringA.substring(i, i + 3).equals(StringB))
                count++;
        }
        System.out.println("StringB occurs " + count + " times in String A.");
    }

//Container containerTest()

    public static void Cin ( String fileName ) throws IOException {
        BufferedWriter out = new BufferedWriter(new FileWriter( fileName ));
        String[] words = {
            "class FileOutIn{",
            "	private String name;",
            "	private int length;",
            "	public FileOutIn(String name, int length){",
            "		this.name = name;",
            "		this.length = length;",
            "	}",
            "}"
        };
        for(String c:words){
            out.write(c);
            out.newLine();
        }	
	    out.close();
    }
    public static void Cout ( String fileName ) throws IOException  {
	String line;
        BufferedReader in = new BufferedReader( new FileReader( fileName ) );
        line = in.readLine(); 
        String text = new String();
        while ( line != null ) {
            System.out.println( line );
            text += line;
            line = in.readLine();
        }
        in.close();
        StringTokenizer tk = new StringTokenizer(text," .\t(),;=[]{}");
        TreeSet st = new TreeSet();
        int num = 0;
        while(tk.hasMoreTokens()){
            st.add(tk.nextToken());
            ++num;
        }
        System.out.println("Num: "+ num);
        System.out.println("Words: ");
        Iterator it=st.iterator();
        while(it.hasNext()) {
            String fruit=(String)it.next();
                System.out.println(fruit);
        }
    }
    public static void containerTest(String FileName) {
        try {
                Cin(FileName);
            Cout(FileName);
        } catch (IOException e) {
        // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }

//Thread threadTest()

    public static void threadTest() {
	// TODO Auto-generated method stub
	    Scanner sc = new Scanner(System.in);
        System.out.println("Input Count:");
        int count = sc.nextInt();
        System.out.println(count);
        System.out.println("main thread starts. ");
        PrintJishu thread1 = new PrintJishu(count);
        PrintOshu thread2 = new PrintOshu(count);
        thread1.start();
        new Thread(thread2,"Thread2").start();
        System.out.println("main thread ends. ");
    }

//mainTest

    //static fun can only call stactic dynamic classes and static functions
    public static void main(String[] args) throws IOException{
        inputTest();
        shapeTest();
        tuxingTest();
        mathTest();
        stringProcess();
        containerTest("C:/Users/Administrator/Desktop/javaTest.txt");
        threadTest();
    }

}

//Thread 

//extends Thread, override run()
class PrintJishu extends Thread {
    private int num;
    public PrintJishu( int num ) {
        this.num = num;
    }
    public void run() {
        System.out.println("new Thread start to Print Jishu: ");
        Random sleepTime = new Random();
        int i = 1;
        while(i<num) {
            System.out.println(i);
            i = i+2;
            try {
                Thread.sleep(sleepTime.nextInt(500));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        System.out.println("PrintJishu thread ends. ");
    }
}
//Interface Runnable, override run(), Inter base class Thread to realize
class PrintOshu implements Runnable {
    private int num;
    public  PrintOshu( int num ) {
        this.num = num;
    }
    public void run() {
        System.out.println("new Thread start to Print Oshu: ");
        int i = 0;
        Random sleepTime = new Random();
        while(i<num) {
            System.out.println(i);
            i = i+2;
            try {
                Thread.sleep(sleepTime.nextInt(500));
            } catch (InterruptedException e) {
                e.printStackTrace();
            } 
        }
        System.out.println("PrintOshu thread ends. ");
    }
}
```

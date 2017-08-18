# 패키지명
org.androidtown.socket

# DataEncrypt.java
package org.androidtown.socket;

import java.security.MessageDigest;
import java.security.DigestInputStream;
import java.io.BufferedInputStream;

class DataEncrypt {


	public DataEncrypt(){
		
	}
	
	public String encrypt(String strData){
		
		MessageDigest m = null;
	    String hash = null;
	    try{
	        m = MessageDigest.getInstance("MD5");
	        m.update(strData.getBytes(),0,strData.length());
	        byte tmp[]= m.digest();
	        StringBuffer sb = new StringBuffer();
	        for(int i=0;i<tmp.length;++i){
	        	sb.append(String.format("%02X",0xff&tmp[i]));
	        }
	        hash = sb.toString();
	    }
	    catch (Exception e){  
	    	e.printStackTrace(); 
	    }
	    return hash;

	}
	

}



# JavaSocketServer.java
package org.androidtown.socket;

import java.awt.Image;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.net.ServerSocket;
import java.net.Socket;
import java.util.Random;
import java.util.Vector;

import javax.swing.ImageIcon;

public class JavaSocketServer {

	public static void main(String[] args) {
		int port = 5001;
		String qr = "send";
		Vector<String> v_id = new Vector<String>();
		v_id.add("1392024");
		v_id.add("1592017");
		v_id.add("1591033");
		v_id.add("1391080");
		
		String id = null;
		String password = "123456";
		String temp_id = null;
		String temp_pw = null;
		
		String check = null;
		String output = null;
		String output2 = null;
		String temp = null;
		
		int count = 0;
		int id_count=0;
		
		DataEncrypt de = new DataEncrypt();
		
		try {
			ServerSocket server = new ServerSocket(port);
			System.out.println("본 서버시작됨 : " + port);
			
			while(true){
				System.out.println("ok");
				Socket socket = server.accept();
				
				//클라이언트로 부터 데이터가 1개오면 input만 
				ObjectInputStream instream = new ObjectInputStream(socket.getInputStream());
				Object input = instream.readObject();  
				System.out.println("클라이언트로부터 받은 데이터(input) : " + input);
				System.out.println("input 끝");
				
				ObjectInputStream instream2 = new ObjectInputStream(socket.getInputStream());
				Object input2 = instream.readObject();  
				System.out.println("클라이언트로부터 받은 데이터(input2) : " + input2);
				System.out.println("input2 끝 ");
				
				
				
				// QR코드일때만 send가 옴
				if(input.toString().equals("send")){
					System.out.println("kk");
					count=1;
				}
				
				
				else if((input.toString()!="send") && (input2.toString()!="send")){
					System.out.println("kk2");
					temp_id = input.toString();
					temp_pw = input2.toString();
					count=2;
				}
				
				
				// 보낸값 받은값 체킹하는 곳
				/*
				else if(input.toString().equals(output2) && input2.toString().equals(temp)){
					System.out.println("보낸값과 받은 값이 같음");
				}
				*/
				
				
				// 로그인 기능
				if(count==2){
					System.out.println(input.toString());
					for(int i=0; i<v_id.size(); i++){
						id=v_id.get(i);
					
					if(id.equals(temp_id)){
						if(password.equals(temp_pw)){
					check = "허가";
					ObjectOutputStream outstream = new ObjectOutputStream(socket.getOutputStream());
					outstream.writeObject(check);
					outstream.flush();
					System.out.println("클라이언트로 보낸 데이터 : " + check);
					id_count=1;	
					/*
					instream.close();
					outstream.close();
					socket.close();
					count = 0;
					*/
							}
						}
					}
					if(id_count==0){
					check = "불허가";
					ObjectOutputStream outstream = new ObjectOutputStream(socket.getOutputStream());
					outstream.writeObject(check);
					outstream.flush();
					System.out.println("클라이언트로 보낸 데이터 : " + check);
						
					
					instream.close();
					outstream.close();
					socket.close();
					count = 0;
					}
					id_count=0;
				}
				
				// 지문인식이 통과되었을때
				if(count==1){
				//String output = "(" + input + ")" + "~.~";
				Vector<String> v = new Vector<String>();
				for(int j=0; v.capacity()>j; j++){
					Random random = new Random();
					int i = random.nextInt(10); // 0~9
					String s = Integer.toString(i);
					v.add(s);
				}
				
				
				//String output = "https://zxing.org/w/chart?cht=qr&chs=170x170&chld=L&choe=UTF-8&chl=MECARD%3AN%3A%EC%95%88%ED%98%95%EC%9A%B0%3BORG%3A%EA%B8%B0%EC%97%85%EC%9D%80%ED%96%89%3BTEL%3A"+v.get(0)+v.get(1)+"%3BURL%3Ahttp%5C%3A%2F%2Fwww.daum.net%3BEMAIL%3Adksguddn%40daum.net%3BADR%3A%EC%84%9C%EC%9A%B8%EC%8B%9C+%EA%B4%91%EC%A7%84%EA%B5%AC+%EA%B5%AC%EC%9D%98%EB%8F%99+"+v.get(2)+"01%ED%98%B8%3BNOTE%3A%EA%B0%80%EB%82%98%EB%8B%BC%3B%3F";
				output2 = "https://zxing.org/w/chart?cht=qr&chs=170x170&chld=L&choe=UTF-8&chl=MECARD%3AN%3Ahansung%3BORG%3Aunit%3BTEL%3A"+v.get(0)+v.get(1)+"0000000000%3BEMAIL%3Aquestion%40gmail.com%3BADR%3A5897+"+v.get(2)+"301%3BNOTE%3Anotepassword%3B%3B";
				// temp 부분은 MD5로 값 변환시킴
				temp = de.encrypt(output2);
				output = output2 + temp;
			
				//String iutput = "https://zxing.org/w/chart?cht=qr&chs=350x350&chld=L&choe=UTF-8&chl=MECARD%3AN%3AhanKKK%3BORG%3AHansung%3BTEL%3A00099999000%3BEMAIL%3AABC%40naver.com%3BADR%3A%EC%84%9C%EC%9A%B8%EC%8B%9C+%EA%B4%91%EC%A7%84%EA%B5%AC+301%ED%98%B8%3BNOTE%3A%EA%B0%80%EB%82%98%EB%8B%A4%EB%9D%BC%3B%3B";
				ObjectOutputStream outstream = new ObjectOutputStream(socket.getOutputStream());
				outstream.writeObject(output);
				outstream.flush();
				System.out.println("클라이언트로 보낸 데이터 : " + output);
				
				//System.out.println(de.encrypt(output));
				
				instream.close();
				outstream.close();
				socket.close();
				count=0;
				}
			}
		} 
		catch (Exception e) {
		
		}
		
	}

}

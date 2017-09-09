package org.androidtown.socket;

import java.awt.Image;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.net.ServerSocket;
import java.net.Socket;
import java.util.HashMap;
import java.util.Iterator;
import java.util.Random;
import java.util.Set;
import java.util.Vector;

import javax.swing.ImageIcon;

public class JavaSocketServer {

	public static void main(String[] args) {
		
		int port = 80;
		String qr = "ok";
		Vector<String> v_id = new Vector<String>();
		HashMap<String, String> h_md5 = new HashMap<String, String>();
		
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
		String scanner_id = null;
		String scanner_md5 = null;
		String md5 = null;
		String md6 = null;
		String md7 = null;
		
		int count = 0;
		int bigcount = 0;
		int id_count=0;
		int logon_count=0;
		
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
				Object input2 = instream2.readObject();  
				System.out.println("클라이언트로부터 받은 데이터(input2) : " + input2);
				System.out.println("input2 끝 ");
				
				
				/*
				// QR코드일때만 send가 옴
				if(input.toString().equals("send")){
					System.out.println("kk");
					count=1;
				}
				*/
				
				if(input.toString().equals("Scanner")){
					String[] tokenList = input2.toString().split("%3B%3B");
					scanner_id = tokenList[0];
					scanner_md5 = tokenList[2];
					
					ObjectOutputStream outstream;
					
					Set<String> keys = h_md5.keySet();
					Iterator<String> it = keys.iterator();
					
					while(it.hasNext()){
						String key = it.next();
						String value = h_md5.get(key);
						
						// 해쉬맵에 저장된 값과 스캐너에서 받은 값이 같을경우 success리턴 (학번,암호화값)
						if(key.equals(scanner_id)){
							if(value.equals(scanner_md5)){
								outstream = new ObjectOutputStream(socket.getOutputStream());
								outstream.writeObject("success");
								outstream.flush();
								System.out.println("최종 클라이언트로 보낸 데이터 : " + qr);
								outstream.close();
							}
							else{
								outstream = new ObjectOutputStream(socket.getOutputStream());
								outstream.writeObject("fail");
								outstream.flush();
								System.out.println("최종 클라이언트로 보낸 데이터 : " + qr);
								outstream.close();
							}
						}
						
						instream.close();
						instream2.close();
						socket.close();
						
					}
				}
				
				
				else if(input.toString().length() > 30){
					System.out.println("1-1");
					String[] ReturnList = input.toString().split("%3B%3B");
					System.out.println("1-2");
					md5 = ReturnList[0];
					System.out.println("1-3");
					md6 = ReturnList[1];
					System.out.println("1-4");
					md7 = ReturnList[2];
					System.out.println("1-5");
					
					h_md5.put(md5, md7);
				}
				
				/**********중요 구현 부분***********/
				//최종비교 확인이 되는부분
				for(int k=0; k<h_md5.size(); k++){
					if(h_md5.get(k).toString().equals(md7)){
						bigcount=1;   // 최종 비교 확인이 되어 문이 열리는 부분
						System.out.println("2-0");
						ObjectOutputStream outstream = new ObjectOutputStream(socket.getOutputStream());
						outstream.writeObject(qr);
						outstream.flush();
						System.out.println("최종 클라이언트로 보낸 데이터 : " + qr);
					}
					System.out.println("2-1");
				}
				System.out.println("2-2");
				
				
				if((input.toString()!="send") && (input2.toString()!="send")){
					System.out.println("kk2");
					temp_id = input.toString();
					temp_pw = input2.toString();
					
					count=2;
				}
				
				
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
					logon_count=1;
					
					/*
					instream.close();
					outstream.close();
					socket.close();
					count = 0;
					*/
							}
						}
					}
					if(id_count==0 && logon_count==0){
					check = "불허가";
					ObjectOutputStream outstream = new ObjectOutputStream(socket.getOutputStream());
					outstream.writeObject(check);
					outstream.flush();
					System.out.println("클라이언트로 보낸 데이터 : " + check);
						
					
					instream.close();
					instream2.close();
					outstream.close();
					socket.close();
					count = 0;
					}
					id_count=0;
				}
				
				// 지문인식이 통과되었을때
				if(logon_count==1){
				//String output = "(" + input + ")" + "~.~";
				Vector<String> v = new Vector<String>();
				for(int j=0; v.capacity()>j; j++){
					Random random = new Random();
					int i = random.nextInt(10); // 0~9
					String s = Integer.toString(i);
					v.add(s);
				}
				
				
				/*******중요부분********/
				//기존 학번에 대한 암호화값이 들어있으면 덮어씌워야함
				
				if(!check.equals("허가")){
				//String output = "https://zxing.org/w/chart?cht=qr&chs=170x170&chld=L&choe=UTF-8&chl=MECARD%3AN%3A%EC%95%88%ED%98%95%EC%9A%B0%3BORG%3A%EA%B8%B0%EC%97%85%EC%9D%80%ED%96%89%3BTEL%3A"+v.get(0)+v.get(1)+"%3BURL%3Ahttp%5C%3A%2F%2Fwww.daum.net%3BEMAIL%3Adksguddn%40daum.net%3BADR%3A%EC%84%9C%EC%9A%B8%EC%8B%9C+%EA%B4%91%EC%A7%84%EA%B5%AC+%EA%B5%AC%EC%9D%98%EB%8F%99+"+v.get(2)+"01%ED%98%B8%3BNOTE%3A%EA%B0%80%EB%82%98%EB%8B%BC%3B%3F";
				//output2 = "https://zxing.org/w/chart?cht=qr&chs=170x170&chld=L&choe=UTF-8&chl=MECARD%3AN%3Ahansung%3BORG%3Aunit%3BTEL%3A"+v.get(0)+v.get(1)+"0000000000%3BEMAIL%3Aquestion%40gmail.com%3BADR%3A5897+"+v.get(2)+"301%3BNOTE%3Anotepassword%3B%3B";
				output2 = input.toString() + input2.toString();
				// temp 부분은 MD5로 값 변환시킴
				temp = de.encrypt(output2);
				output = output2 + temp;
				//h_md5.put(temp, temp);					//  md5(암호화)된 값을 벡터에 임시저장함
				
				//String iutput = "https://zxing.org/w/chart?cht=qr&chs=350x350&chld=L&choe=UTF-8&chl=MECARD%3AN%3AhanKKK%3BORG%3AHansung%3BTEL%3A00099999000%3BEMAIL%3AABC%40naver.com%3BADR%3A%EC%84%9C%EC%9A%B8%EC%8B%9C+%EA%B4%91%EC%A7%84%EA%B5%AC+301%ED%98%B8%3BNOTE%3A%EA%B0%80%EB%82%98%EB%8B%A4%EB%9D%BC%3B%3B";
				ObjectOutputStream outstream = new ObjectOutputStream(socket.getOutputStream());
				outstream.writeObject(output);
				outstream.flush();
				System.out.println("클라이언트로 보낸 데이터3 : " + output);
				
				//System.out.println(de.encrypt(output));
				
				instream.close();
				instream2.close();
				outstream.close();
				socket.close();
				count=0;
				}
			
				}
				check="check";
				System.out.println(count);
			}
		} 
		catch (Exception e) {
			
		}
	
		
	}

}







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


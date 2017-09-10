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
import java.awt.*;

import javax.swing.ImageIcon;

public class JavaSocketServer {

	public static void main(String[] args) {
		
		ImageIcon img = new ImageIcon("C:\\back.jpg");
		Image sendimg = img.getImage();
		
		int port = 80;
		String qr = "ok";
		//Vector<String> v_id = new Vector<String>();
		HashMap<String, Login> login_id = new HashMap<String, Login>();
		//HashMap<String, String> h_md5 = new HashMap<String, String>();
		HashMap<String, String> h_md5 = new HashMap<String, String>();
		
		login_id.put("1392024", new Login("123456", "안형우"));
		login_id.put("1592017", new Login("123456", "송정은"));
		login_id.put("1591033", new Login("123456", "진소린"));
		login_id.put("1391080", new Login("123456", "한경동"));
		
		Set<String> login_id_keys;
		Iterator<String> login_id_iterator;
		
		
		String temp_id = null;
		String temp_pw = null;
		
		String check = "불허가";
		String output = null;
		String output2 = null;
		String output_id = null;
		String output_name = null;
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
					
					login_id_keys = login_id.keySet();
					login_id_iterator = login_id_keys.iterator();
				
					while(login_id_iterator.hasNext()){
						String id_key = login_id_iterator.next();
						String id_value = login_id.get(id_key).pw;
						System.out.println("아이디 : " + temp_id);	
						if(id_key.equals(temp_id)){
							System.out.println("비번 : " + temp_pw);
							if(id_value.equals(temp_pw)){
								check = "허가";
								ObjectOutputStream outstream = new ObjectOutputStream(socket.getOutputStream());
								outstream.writeObject(check);
								outstream.flush();
								System.out.println("클라이언트로 보낸 데이터 : " + check);
								id_count=1;	
								logon_count=1;
								outstream.close();
								
										}
									}
								}
								if(id_count==0 && logon_count==0){
								check = "불허가";
								ObjectOutputStream outstream = new ObjectOutputStream(socket.getOutputStream());
								outstream.writeObject(check);
								outstream.flush();
								System.out.println("클라이언트로 보낸 데이터 : " + check);
									
								count = 0;
								outstream.close();
								instream.close();
								instream2.close();
								socket.close();
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
				output2 = input.toString() + input2.toString();
				// temp 부분은 MD5로 값 변환시킴
				temp = de.encrypt(output2);
				output = output2 + temp;
				//h_md5.put(temp, temp);					//  md5(암호화)된 값을 벡터에 임시저장함
				
				login_id_keys = login_id.keySet();
				login_id_iterator = login_id_keys.iterator();
			
				while(login_id_iterator.hasNext()){
					String id_key = login_id_iterator.next();
					String id_value = login_id.get(id_key).pw;
					System.out.println("output아이디 : " + temp_id);
					String id_key_temp = id_key + "%3B%3B";
					if(id_key_temp.equals(temp_id)){
						System.out.println("output준비완료 : " + temp_id);
							output_id = id_key;
							output_name = login_id.get(id_key).name;
							
						}
					}
						
				
				
				
				
				
				ObjectOutputStream outstream = new ObjectOutputStream(socket.getOutputStream());
				outstream.writeObject(output);
				outstream.flush();
				System.out.println("클라이언트로 보낸 QR데이터 : " + output);
				
				ObjectOutputStream outstream2 = new ObjectOutputStream(socket.getOutputStream());
				outstream2.writeObject(output_id);
				outstream2.flush();
				System.out.println("클라이언트로 보낸 학번 데이터 : " + output_id);
				//System.out.println(de.encrypt(output));
				
				ObjectOutputStream outstream3 = new ObjectOutputStream(socket.getOutputStream());
				outstream3.writeObject(output_name);
				outstream3.flush();
				System.out.println("클라이언트로 보낸 이름 데이터 : " + output_name);
				//System.out.println(de.encrypt(output));
				
				instream.close();
				instream2.close();
				outstream.close();
				outstream2.close();
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








package org.androidtown.socket;

public class Login {
	String pw;
	String name;
	
	public Login(){
		this.pw = "?";
		this.name = "??";
	}
	public Login(String pw, String name){
		this.pw = pw;
		this.name = name;
	}
}


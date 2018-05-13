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

		//String path = "C:\\Users\\hscom-017\\Desktop\\img";
		
		String qr = "ok";
		HashMap<String, Login> login_id = new HashMap<String, Login>();
		HashMap<String, String> h_md5 = new HashMap<String, String>();

		login_id.put("1392024", new Login("123456", "안형우", "01041199582"));
		login_id.put("1592017", new Login("123456", "송정은", "01055528461"));
		login_id.put("1591033", new Login("123456", "진소린", "01074771488"));
		login_id.put("1391080", new Login("123456", "한경동", "01066900694"));
		login_id.put("1392080", new Login("123456", "최용석", "01082229352"));

		Set<String> login_id_keys;
		Iterator<String> login_id_iterator;

		String temp_id = null;
		String temp_pw = null;
		String temp_phone = null;

		String check = null;
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
		int id_count;
		int logon_count;
		int scanner_count = 0;

		DataEncrypt de = new DataEncrypt();

		try {
			ServerSocket server = new ServerSocket(port);
			System.out.println("본 서버시작됨 : " + port);

			while (true) {
				System.out.println("server restart ok");
				Socket socket = server.accept();

				check="모름";
				id_count=0;
				logon_count=0;
				
				
				// 클라이언트로 부터 데이터가 1개오면 input만
				ObjectInputStream instream = new ObjectInputStream(socket.getInputStream());
				Object input = instream.readObject();
				System.out.println("클라이언트로부터 받은 데이터(input) : " + input);
				System.out.println("input 끝");

				ObjectInputStream instream2 = new ObjectInputStream(socket.getInputStream());
				Object input2 = instream2.readObject();
				System.out.println("클라이언트로부터 받은 데이터(input2) : " + input2);
				System.out.println("input2 끝 ");

				// 입력받은 값 2개가 길이가 68이상이면 QR코드 액티비티에서 보낸값인걸로 알고 수행
				String concat = input.toString() + input2.toString();

				System.out.println("concat 끝");

					
				//NFC 데이터 부분
				if(input.toString().equals("nfc")){
					//String nfc_check = nfc.datacheck(concat);
				
					check = nfc.datacheck(concat);
					System.out.println("NFCNFCNFC : " + check);
				
					ObjectOutputStream outstream = new ObjectOutputStream(socket.getOutputStream());
					outstream.writeObject(check);
					outstream.flush();
					
					id_count = 1;
					logon_count = 1;
					instream.close();
					instream2.close();
					outstream.close();
				
				/*	ObjectOutputStream outstream = new ObjectOutputStream(socket.getOutputStream());
					outstream.writeObject(check);
					outstream.flush();
				
					id_count = 1;
					logon_count = 1;
					instream.close();
					instream2.close();
					outstream.close();	*/
				
			}
				
				// 스캐너에서 데이터를 보낼때 처리하는 부분
				if (input.toString().equals("scanner")) {
					String compare = "%3B%3B";
					String result;

					System.out.println("스캐너 카운터 before");
					for (int p = 0; p < input2.toString().length() - 6; p++) {
						result = input2.toString().substring(p, p + 6);
						System.out.println(p + "  " + result);
						if (compare.equals(result)) {
							scanner_count = 1;
							break;
						}
					}

					if (scanner_count != 1) {
						ObjectOutputStream scanner_outstream = new ObjectOutputStream(socket.getOutputStream());
						scanner_outstream.writeObject("fail");
						scanner_outstream.flush();
						System.out.println("최종 클라이언트로 보낸 데이터 : " + "fail");

						scanner_outstream.close();
						instream.close();
						instream2.close();

					}

					else if (scanner_count == 1) {
						System.out.println("input2 - " + input2);
						String[] tokenList = input2.toString().split("%3B%3B");
						scanner_id = tokenList[0];
						System.out.println(tokenList[1]);
						scanner_md5 = tokenList[2];
						System.out.println("스캐너 학번 : " + scanner_id);
						System.out.println("스캐너 암호 : " + scanner_md5);

						ObjectOutputStream outstream;

						//QR코드를 보낼때 저장된 학번과 암호화값과 스캐너에서 보낸 값들과 비교
						Set<String> keys = h_md5.keySet(); 
						Iterator<String> it = keys.iterator();

						if(it.hasNext()==false){
								outstream = new ObjectOutputStream(socket.getOutputStream());
								outstream.writeObject("fail");
								outstream.flush();
								System.out.println("최종 클라이언트로 보낸 데이터 : " + "fail");
								outstream.close();
						}
						
						while (it.hasNext()) {
							//System.out.println("success 전 + it.next 값 - " + it.next());
							String key = it.next();
							System.out.println("success 전 + key 값 - " + key);
							String value = h_md5.get(key);
							System.out.println("success 전  + value 값 - " + value);
							System.out.println("aaaa");
							// 해쉬맵에 저장된 값과 스캐너에서 받은 값이 같을경우 success리턴 (학번,암호화값)
							if (key.equals(scanner_id)) {
								if (value.equals(scanner_md5)) {
									outstream = new ObjectOutputStream(socket.getOutputStream());
									outstream.writeObject("success");
									outstream.flush();
									System.out.println("최종 클라이언트로 보낸 데이터 : " + "success");
									//instream.close();
									//instream2.close();
									outstream.close();
									System.out.println("success 후 1");
									// 쓴 키값은 삭제해버림
									h_md5.remove(key);
									break;
									
									//System.out.println("success 후 2 : 해쉬 삭제");
								} 
								else {
									outstream = new ObjectOutputStream(socket.getOutputStream());
									outstream.writeObject("fail");
									outstream.flush();
									System.out.println("최종 클라이언트로 보낸 데이터 : " + "fail");
									//instream.close();
									//instream2.close();
									outstream.close();
								}
								System.out.println("success 후 3 ");
							}
						}
						System.out.println("success 후 4 : instream 1 클로즈 ");
						instream.close();
						System.out.println("success 후 5 : instream 2 클로즈 ");
						instream2.close();
						//socket.close();
						
						//여기서 삭제해야됨
						
					}
					System.out.println("success 후 6 : 스캐너 카운터 0 ");
					scanner_count = 0;
				}

				/*
				else if (concat.toString().length() > 68) {
					System.out.println("콘캣들어옴");
					String[] ReturnList = concat.toString().split("%3B%3B");

					md5 = ReturnList[0]; // 학번
					md6 = ReturnList[1]; // 날짜
					md7 = ReturnList[2]; // md5암호화값
					
					System.out.println("MD55555555 : " + md5.toString());
					System.out.println("MD66666666 : " + md6.toString());
					System.out.println("MD77777777 : " + md7.toString());
					// if문 -> 이미 md5, 즉 해당 학번에 해당하는 암호값이 있다면 변경
					h_md5.put(md5, md7);
				}
				*/

				
				
				// 첫 로그인화면 '허가' or '불허가' 체크하는 부분
				if (!(input.toString().equals("send")) && !(input2.toString().equals("send"))
						&& !(input.toString().equals("scanner")) && !(input.toString().equals("imgcall"))) 
				{
					System.out.println("kk2");
					temp_id = input.toString();
					System.out.println("kk2_1");
					//토큰 자르는거 추가 (비밀번호, 전화번호)
					String[] loginlist = input2.toString().split("%3B%3B");
					temp_pw = loginlist[0];
					temp_phone = loginlist[1];
					//temp_pw = input2.toString();
					System.out.println("kk3");
				
					// 로그인 기능
					login_id_keys = login_id.keySet();
					login_id_iterator = login_id_keys.iterator();
					System.out.println("kk4");
					
					while (login_id_iterator.hasNext()) {
						String id_key = login_id_iterator.next();
						String id_value = login_id.get(id_key).pw;
						String id_value2 = login_id.get(id_key).phone;
						System.out.println("아이디 : " + temp_id);
						if (id_key.equals(temp_id)) {
							System.out.println("비번 : " + temp_pw);
							System.out.println("전화번호" + temp_phone);
							if (id_value.equals(temp_pw) && id_value2.equals(temp_phone)) {
								check = "허가";
								ObjectOutputStream outstream = new ObjectOutputStream(socket.getOutputStream());
								outstream.writeObject(check);
								outstream.flush();
								System.out.println("클라이언트로 보낸 데이터 : " + check);
								
								//OutputImg opi = new OutputImg(socket);
								//opi.output(temp_id);
								
								
								id_count = 1;
								logon_count = 1;
								instream.close();
								instream2.close();
								outstream.close();

							}
						}
					}

					// 최초로그인시 데이터에 없는 학번으로 로그인 할려할때
					if (id_count == 0 && logon_count == 0) {
						check = "불허가";
						ObjectOutputStream outstream = new ObjectOutputStream(socket.getOutputStream());
						outstream.writeObject(check);
						outstream.flush();
						System.out.println("클라이언트로 보낸 데이터 : " + check);

						count = 0;
						instream.close();
						instream2.close();
						outstream.close();
					}

					id_count = 0;
				}
				
				
				
				// 이미지 보내는 부분
				if(input.toString().equals("imgcall")){
					check = "이미지콜 중";
					ObjectOutputStream outstream = new ObjectOutputStream(socket.getOutputStream());
					outstream.writeObject(check);
					outstream.flush();
					System.out.println("클라이언트로 보낸 데이터 : " + check);
					
					System.out.println("이미지콜 시작");
					
			
					OutputImg opi = new OutputImg(socket);
					opi.output(temp_id);
					
					System.out.println("이미지콜 끝");
					
					instream.close();
					instream2.close();
					outstream.close();
				}
				
				
				
				
				// 위 메인액티비티와 이미지부분이 안돌아가면 이 아래코드가 돌아감 == QR코드값 부분
				/******* 중요부분 ********/
				// 기존 학번에 대한 암호화값이 들어있으면 덮어씌워야함
				System.out.println("1-0");
				if ((check.equals("모름")) && !input.toString().equals("scanner")) { 
					
					//output2 = input.toString() + input2.toString();
					output2=input2.toString();
					
					System.out.println("들어온 input데이터 : " + input);
					System.out.println("들어온 input2데이터 : " + input2);
					//9자리 랜덤숫자를 만들어 md5암호화할때 넣어줌(경우의수 10^9)
					String s = ""; 
					
					Random random = new Random();
					
					for(int md5_random=0; md5_random<9; md5_random++){
						s = s + Integer.toString(random.nextInt(10));
					}
						
					
					// temp 부분은 MD5로 값 변환시킴
					temp = de.encrypt(output2+s);
					//output = output2 + "%3B%3B" + temp;
					//output = temp;
					output = output2 + "%3B%3B" + temp;
					
					String[] ReturnList2 = output2.toString().split("%3B%3B");

					String output_id2 = ReturnList2[0]; // 학번
					String output_date2 = ReturnList2[1]; // 날짜
					//String output_md5 = ReturnList2[2]; // md5암호화값

					
					
					login_id_keys = login_id.keySet();
					login_id_iterator = login_id_keys.iterator();

					while (login_id_iterator.hasNext()) {
						String id_key = login_id_iterator.next();
						String id_value = login_id.get(id_key).pw;
						System.out.println("output아이디 : " + output_id2);
						String id_key_temp = id_key;
						if (id_key_temp.equals(output_id2)) {
							System.out.println("output준비완료 : " + output_id2);
							output_id = id_key;
							output_name = login_id.get(id_key).name;

							//OutputImg opi = new OutputImg(socket);
							//opi.output();
							
						}
					}
					System.out.println("1-1");
					h_md5.put(output_id, temp); // md5(암호화)된 값을 벡터에 임시저장함
					System.out.println("1-2");
					ObjectOutputStream outstream = new ObjectOutputStream(socket.getOutputStream());
					outstream.writeObject(output);
					outstream.flush();
					System.out.println("클라이언트로 보낸 QR데이터 : " + output);
					System.out.println("1-3");
					ObjectOutputStream outstream2 = new ObjectOutputStream(socket.getOutputStream());
					outstream2.writeObject(output_id);
					outstream2.flush();
					System.out.println("클라이언트로 보낸 학번 데이터 : " + output_id);
					System.out.println("1-4");
					ObjectOutputStream outstream3 = new ObjectOutputStream(socket.getOutputStream());
					outstream3.writeObject(output_name);
					outstream3.flush();
					System.out.println("클라이언트로 보낸 이름 데이터 : " + output_name);
					System.out.println("1-5");
					
					/*
					OutputImg opi = new OutputImg(socket);
					opi.output();
					*/
					
					instream.close();
					instream2.close();
					outstream.close();
					outstream2.close();
					outstream3.close();
					//socket.close();
					count = 0;
				}
			}// <-- while문 끝나는 부분

		} catch (Exception e) {
			System.out.println("오류");
			e.printStackTrace();
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
	String phone;
	
	public Login(){
		this.pw = "?";
		this.name = "??";
		this.phone = "???";
	}
	public Login(String pw, String name, String phone){
		this.pw = pw;
		this.name = name;
		this.phone = phone;
	}
}








package org.androidtown.socket;


import java.io.BufferedInputStream;
import java.io.BufferedOutputStream;
import java.io.DataOutputStream;
import java.io.File;
import java.io.FileInputStream;
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


public class OutputImg {
	String path = "C:\\Users\\hscom-017\\Desktop\\img";
	Socket socket;
	
	public OutputImg(Socket socket){
		this.socket = socket;
	}
	
	public void output(String temp_id){
		String id = temp_id + ".jpg";
		File[] files = new File(path).listFiles(); // path경로에있는 파일 모두 읽어들임

		try { 
			BufferedOutputStream bos = new BufferedOutputStream(socket.getOutputStream());

			DataOutputStream dos = new DataOutputStream(bos);

			//dos.writeInt(files.length);  //파일 갯수 출력합니다.
			//System.out.println("경로에 있는 총 파일 갯수 : " + files.length);
			
		
			dos.writeInt(1);  //보낼 파일 갯수 입력합니다.   -- 여기선 한개만 보내겠다.
			
			for (File file : files) {
			
			
				String name = file.getName();	 

				if(id.equals(name)){
					long length = file.length();
					dos.writeLong(length);    //파일 길이 출력합니다. 
					System.out.println();

				System.out.println("보낸파일 = " + file.getName()); //파일 이름 출력합니다.
				dos.writeUTF(name);

				FileInputStream fis = new FileInputStream(file);
				BufferedInputStream bis = new BufferedInputStream(fis);	 

				int theByte = 0;

				while ((theByte = bis.read()) != -1) // BufferedInputStream으로// 클라이언트에 보내기 위해 write함니다.

				{
					// System.out.println(file.getName()+"보냄");
					bos.write(theByte);	
				}

				//bos.flush();

				System.out.println("송신완료");				

				//bis.close();
				break;
				}
			}
			dos.flush();	 

		} catch (Exception e) {

			e.printStackTrace();

		}
	}
	
}








import java.io.ObjectOutputStream;
import java.util.HashMap;
import java.util.Iterator;
import java.util.Set;

public class NFCCheck {
	HashMap<String, Login> login_id = new HashMap<String, Login>();
	Set<String> login_id_keys;
	Iterator<String> login_id_iterator;
	
	String check = null;
	
	public NFCCheck(HashMap<String, Login> login_id){
		this.login_id = login_id;
		this.login_id_keys = login_id.keySet();
		this.login_id_iterator = login_id_keys.iterator();
	}
	
	public String datacheck(String nfcdata){
		
		this.login_id_keys = login_id.keySet();
		this.login_id_iterator = login_id_keys.iterator();
		
		String nfcdata_list[] = nfcdata.split("%3B%3B");
		
		// 0번째 - nfc , 1번째 - nfcData , 2번째 - nfcTagId , 3번째 - 학번(id) , 4번째 - password , 5번째 - 전화번호  
		for(int i=0; i<nfcdata_list.length; i++){
			System.out.println("NFC_Check - " + i + "번째 : " + nfcdata_list[i].toString());
		}
	
		
		while (login_id_iterator.hasNext()) {
			String id_key = login_id_iterator.next();
			String id_value = login_id.get(id_key).pw;
			String id_value2 = login_id.get(id_key).phone;
			System.out.println("nfc아이디 : " + nfcdata_list[3]);
			if (id_key.equals(nfcdata_list[3])) {
				System.out.println("nfc비번 : " + nfcdata_list[4]);
				System.out.println("nfc전화번호 :" + nfcdata_list[5]);
				if (id_value.equals(nfcdata_list[4]) && id_value2.equals(nfcdata_list[5])){
					System.out.println("nfc태그 :" + nfcdata_list[2]);
					if("36113665213982980".equals(nfcdata_list[2])){
						check = "nfcOK";
						break;
					}
					else{
						check = "nfcNOK";
						break;
					}
				}
				
			}else{
				check = "불허가";
			}
		}
		//System.out.println("nfc이터레이트 : " + login_id_iterator.next().toString());
		return check;
	}
}


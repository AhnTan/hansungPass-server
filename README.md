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
		HashMap<String, Login> login_id = new HashMap<String, Login>();
		HashMap<String, String> h_md5 = new HashMap<String, String>();

		login_id.put("1392024", new Login("123456", "안형우", "01041199582"));
		login_id.put("1592017", new Login("123456", "송정은", "01055528461"));
		login_id.put("1591033", new Login("123456", "진소린", "01074771488"));
		login_id.put("1391080", new Login("123456", "한경동", "01066900694"));

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
						String[] tokenList = input2.toString().split("%3B%3B");
						scanner_id = tokenList[0];
						System.out.println(tokenList[1]);
						scanner_md5 = tokenList[2];
						System.out.println("스캐너 학번 : " + scanner_id);
						System.out.println("스캐너 암호 : " + scanner_md5);

						ObjectOutputStream outstream;

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
							String key = it.next();
							String value = h_md5.get(key);
							System.out.println("aaaa");
							// 해쉬맵에 저장된 값과 스캐너에서 받은 값이 같을경우 success리턴 (학번,암호화값)
							if (key.equals(scanner_id)) {
								if (value.equals(scanner_md5)) {
									outstream = new ObjectOutputStream(socket.getOutputStream());
									outstream.writeObject("success");
									outstream.flush();
									System.out.println("최종 클라이언트로 보낸 데이터 : " + "success");
									instream.close();
									instream2.close();
									outstream.close();

									// 쓴 키값은 삭제해버림
									h_md5.remove(key);
								} else {
									outstream = new ObjectOutputStream(socket.getOutputStream());
									outstream.writeObject("fail");
									outstream.flush();
									System.out.println("최종 클라이언트로 보낸 데이터 : " + "fail");
									instream.close();
									instream2.close();
									outstream.close();
								}
							}
						}
						instream.close();
						instream2.close();
						socket.close();

					}
					scanner_count = 0;
				}

				else if (concat.toString().length() > 68) {
					System.out.println("콘캣들어옴");
					String[] ReturnList = concat.toString().split("%3B%3B");

					md5 = ReturnList[0]; // 학번
					md6 = ReturnList[1]; // 날짜
					md7 = ReturnList[2]; // md5암호화값

					// if문 -> 이미 md5, 즉 해당 학번에 해당하는 암호값이 있다면 변경
					h_md5.put(md5, md7);
				}

				if (!(input.toString().equals("send")) && !(input2.toString().equals("send"))
						&& !(input.toString().equals("scanner"))) {
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
					output = output2 + "%3B%3B" + temp;
					//output = temp;
					
					
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
					instream.close();
					instream2.close();
					outstream.close();
					outstream2.close();
					//socket.close();
					count = 0;
				}
			}

		} catch (Exception e) {

		}
	}
}

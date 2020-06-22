# Soham-s-project
#Check current car value

package sohamJanaCarCheck;
import java.io.BufferedReader;
import java.io.File;
import java.io.FileNotFoundException;
import java.io.FileReader;
import java.io.IOException;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.util.ArrayList;
import java.util.regex.Matcher;
import java.util.*;
import java.util.regex.Pattern;

import org.openqa.selenium.By;
import org.openqa.selenium.NoAlertPresentException;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.firefox.FirefoxDriver;
import java.util.concurrent.TimeUnit;

public class carCheck {

	WebDriver driver;
	
	/*extract license info from input.txt*/
	public List<String> readNExtractVehicleRegInfo() {
		Path currentDir = Paths.get(".");		
		List<String> reginfoList = new ArrayList<String>(); 
		File file = new File(currentDir+"\\FW__New_test\\car_input.txt"); 
		  
		  BufferedReader br=null;
		try {
			br = new BufferedReader(new FileReader(file));
		} catch (FileNotFoundException e) {			
			e.printStackTrace();
		} 
		  
		  String st; 
		  try {
//			  int count = 0;
			while ((st = br.readLine()) != null) {			
			    String regex = "[A-Za-z]{2}[0-9]{2}\\s*[A-Za-z]{3}";
			    Pattern pattern = Pattern.compile(regex);
			    Matcher matcher = pattern.matcher(st);	    
			    while(matcher.find()) {
					/*
					 * count++; System.out.println("found: " + count + " : " + matcher.start() +
					 * " - " + matcher.end()+":::"+ st.substring(matcher.start(),
					 * matcher.end()).replaceAll("\\s","") );
					 */
			        reginfoList.add(st.substring(matcher.start(), matcher.end()).replaceAll("\\s",""));
			    }
			    }
		} catch (IOException e) {
			System.out.println(e.toString());
			e.printStackTrace();
		} 
		return reginfoList;
	}
	
	public boolean isAlertPresent() 
	{ 
	    try 
	    { 
	    	this.driver.switchTo().alert(); 
	        return true; 
	    }  
	    catch (NoAlertPresentException Ex) 
	    { 
	        return false; 
	    }   
	}   
	
	
	/*Read the information from the output.txt  file and prepare a list of hash AKA 
	 * linkedhashmap with registration as key*/
	public LinkedHashMap<String, List<String>> vehicleInfo() {
		LinkedHashMap<String, List<String>> lhm = 
                new LinkedHashMap<String, List<String>>();
		Path currentDir = Paths.get(".");			
		File file = new File(currentDir+"\\FW__New_test\\car_output.txt"); 
		
		  
		  BufferedReader br=null;
		try {
			br = new BufferedReader(new FileReader(file));
		} catch (FileNotFoundException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} 
		
		String st;
		
		try {
			while ((st = br.readLine()) != null) {			
			    String regex = "[A-Za-z]{2}[0-9]{2}\\s*[A-Za-z]{3}";/*make sure that the lines have reg info*/
			    Pattern pattern = Pattern.compile(regex);
			    Matcher matcher = pattern.matcher(st);	    
			    while(matcher.find()) {
			    	String[] arrOfStr = st.split(",", 0);			    	
			      lhm.put(st.substring(matcher.start(), matcher.end()).replaceAll("\\s",""), Arrays.asList(arrOfStr));
			    }
			    }
			
		} catch (IOException e) {
			System.out.println(e.toString());
			e.printStackTrace();
		}
		
		return lhm;
	}
	
	public void compareInfo(List<String> outputFileinfo,List<String> webInfo) {
		
		if(outputFileinfo.size() != webInfo.size()) {
			System.out.println("Information size mismatch for: "+ outputFileinfo.get(0));
			return;
		}
		for (int i = 0; i < outputFileinfo.size(); i++) {  
			System.out.println("comparing Information mismatch for: "+ outputFileinfo.get(i).toLowerCase() + " "+ webInfo.get(i).toLowerCase());
          if(  outputFileinfo.get(i).toLowerCase().compareTo(webInfo.get(i).toLowerCase()) !=0) {
        	  System.out.println("Information mismatch for: "+ outputFileinfo.get(i).toLowerCase() + " "+ webInfo.get(i).toLowerCase());
          }
        }
	}
	
	public static void main(String[] args) {
		carCheck testObj = new carCheck();
		
		LinkedHashMap<String, List<String>>vehicleInfofromOutputfile = testObj.vehicleInfo();
		
		List<String> reginfoList = testObj.readNExtractVehicleRegInfo();
		
		 System.setProperty("webdriver.gecko.driver","C:\\geckodriver-v0.26.0-win64\\geckodriver.exe");
		 testObj.driver = new FirefoxDriver();
		 String baseUrl = "https://cartaxcheck.co.uk/";
		 Iterator<String> reginfoIterator = reginfoList.iterator();
		
		 while(reginfoIterator.hasNext()) { 
			 List<String> webInfo = new ArrayList<String>();
			 testObj.driver.get(baseUrl);
			 String reg = reginfoIterator.next();
			 System.out.println("\n checking info for: " + reg );
			 testObj.driver.findElement(By.id("vrm-input")).sendKeys(reg);//input reginfo
			 testObj.driver.findElement(By.xpath("//button[text()=\"Free Car Check\"]")).click();
			 testObj.driver.manage().timeouts().implicitlyWait(10,TimeUnit.SECONDS);
			 try {
				 Thread.sleep(5000);
				 } catch (InterruptedException e){ // TODO Auto-generated catch block
					 e.printStackTrace();
					 }
			 if(testObj.isAlertPresent()) {
				 System.out.println("No information found for : " +reg);
				 continue;
				 }
			// System.out.println("checking info for: " +testObj.driver.findElement(By.xpath("//*[text()=\"Registration\"]/following-sibling::*")).getText());
			// System.out.println("checking info for: " +testObj.driver.findElement(By.xpath("//*[text()=\"Make\"]/following-sibling::*")).getText());
			// System.out.println("checking info for: " +testObj.driver.findElement(By.xpath("//*[text()=\"Model\"]/following-sibling::*")).getText());
			// System.out.println("checking info for: " +testObj.driver.findElement(By.xpath("//*[text()=\"Colour\"]/following-sibling::*")).getText());
			// System.out.println("checking info for: " +testObj.driver.findElement(By.xpath("//*[text()=\"Year\"]/following-sibling::*")).getText());
			 
			 webInfo.add(testObj.driver.findElement(By.xpath("//*[text()=\"Registration\"]/following-sibling::*")).getText().trim());
			 webInfo.add(testObj.driver.findElement(By.xpath("//*[text()=\"Make\"]/following-sibling::*")).getText().trim());
			 webInfo.add(testObj.driver.findElement(By.xpath("//*[text()=\"Model\"]/following-sibling::*")).getText().trim());
			 webInfo.add(testObj.driver.findElement(By.xpath("//*[text()=\"Colour\"]/following-sibling::*")).getText().trim());
			 webInfo.add(testObj.driver.findElement(By.xpath("//*[text()=\"Year\"]/following-sibling::*")).getText().trim());
			 if(vehicleInfofromOutputfile.containsKey(reg)) {
					testObj.compareInfo(vehicleInfofromOutputfile.get(reg),webInfo);
				}else {
					System.out.println("output file doesn't have info for : " + reg);
				}
		 }
		  
		  
		  testObj.driver.close();
		
	
	}

}

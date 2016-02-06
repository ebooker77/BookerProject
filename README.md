# BookerProject
ReadLith

import processing.core.*;
import edu.duke.*;
import java.io.*;
import java.util.*;
import java.lang.*;

public class ReadLith {
	FileResource lith;
	File lithFile; 
	File wellFile;
	ArrayList<String> lithLines;
	ArrayList<String> lithSym;
	LinkedHashMap<Character, String> lithSymMap;
	FileResource litCFG; // 
	LinkedHashMap<Float, String> lithDataMap;
	String lithHeader;
	String liths;
	ArrayList<Float> lithDepth;
	
	public ReadLith(File lithFile) {
		this.lithFile = lithFile;
		lith = new FileResource(lithFile);
		//wellFile = new File("C:/Users/Eric/SkyDrive/data");  // Desktop location
		wellFile = new File("C:/Users/EBooker/OneDrive/data"); // Laptop location
		lithLines = new ArrayList<String>();
		lithSym = new ArrayList<String>();
		lithSymMap = new LinkedHashMap<Character, String>();
		litCFG = new FileResource(wellFile + "/litcfg.txt"); 
		lithDataMap = new LinkedHashMap<Float, String>();
		lithDepth = new ArrayList<Float>();
	}

	public void getData() {
		
		lithLines = fillArray(lith);
		lithSym = fillArray(litCFG);
		lithSymMap = fillHashMap(lithSym);
		lithDataMap = getLithData(lithLines);
		
		for (String s : lithSym) {
			
		}
		
		
		printData();
	}
	
	public void printData() {
		System.out.println(lithHeader);
				
		System.out.println();
		
		System.out.println("Lithology:");
		for(Float f : lithDepth)  System.out.println(f.intValue() + // Convert float to int.
				"\t" + lithDataMap.get(f));  
			
		System.out.println(lithHeader);
	}
		
	public LinkedHashMap<Float, String> getLithData(ArrayList<String> al) {  
		LinkedHashMap<Float, String> hm = new LinkedHashMap<Float, String>();
		Float depth = 0f;
		String liths;
		int delim;   //delim = index of first delimiter which is a space
		String header = new String();
				
		for (int i = 0; i < al.size(); i++) {
			
			delim = al.get(i).indexOf((char)32);      //32 is ascii for " " (space).
			liths = al.get(i).substring(delim);
			
			if (!isNumeric(al.get(i).substring(0, delim))){ //Skips lines that doesn't start with a number
				header = al.get(i).substring(0);  
				hm.put(999999f, header);          //assumes line that doesn't begin with a number is the header title.
				lithHeader = header;
				hm.remove(999999f);
				continue;                         // and stores it in header variable 
			}
			
			depth = Float.valueOf(al.get(i).substring(0, delim));
			lithDepth.add(depth);      // Adds depth only to arrayList listDepth
			hm.put(depth, liths);      //fills the hashmap with depths as keys and liths as values.
			
		}
		
		return hm;
	}
	
	public ArrayList<String> fillArray(FileResource file) {
		ArrayList<String> al = new ArrayList<String>();
		for (String line : file.lines()) {
			if (line.length() > 0) {
				al.add(line);
			}
		}
		return al;
	}
	
	public LinkedHashMap<Character, String> fillHashMap (ArrayList<String> al) {
		LinkedHashMap<Character, String> hm = new LinkedHashMap<Character, String>();
		
		for (int i = 0; i < al.size(); i++) {
			char c = al.get(i).charAt(0);  // first character of line. Represents symbol
			int j = al.get(i).indexOf(c) + 2;
			String s = al.get(i).substring(j);  //al is lithSym
			//System.out.print(c + ", ");
			//System.out.println(s);
			hm.put(c, s);  // hm is lithSymMap
		}
		
		return hm;
		
	}
	
	 // Method found on Stack Overflow. Checks to see if a String is a number.
	 public static boolean isNumeric(String str)  
	  {
	    try
	    {
	      double d = Double.parseDouble(str);
	    }
	    catch(NumberFormatException nfe)
	    {
	      return false;
	    }
	    return true;
	  }
	 	
	private void out(String s){ //Shortcut method for printing to the console
		System.out.println(s);		
	}
	
	public static void main(String[] args) {
		//File lithtxt = new File("C:/Users/Eric/oneDrive/data/lith.txt");    // Desktop locatio
		File lithtxt = new File("C:/Users/EBooker/OneDrive/data/lith.txt");   // Laptop location
		ReadLith test = new ReadLith(lithtxt);
		test.getData();
	}

}

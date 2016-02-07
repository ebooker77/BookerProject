# BookerProject
ReadLith
import processing.core.*;
import edu.duke.*;
import java.io.*;
import java.util.*;

import javax.swing.*;

public class ReadLith {
	FileResource lith;
	File lithFile; 
	File wellFile;
	ArrayList<String> lithLines;
	ArrayList<String> lithKey;
	LinkedHashMap<Character, String> lithSymMap;
	FileResource litCFG; // 
	LinkedHashMap<Float, String> lithDataMap;
	String lithHeader;
	String liths;
	ArrayList<Float> lithDepth;
	ArrayList<Character> lithSym;
	public ArrayList<String> percentages;
	
	public ReadLith(File lithFile) {
		this.lithFile = lithFile;
		lith = new FileResource(lithFile);
		wellFile = new File("C:/Users/Eric/SkyDrive/data");
		lithLines = new ArrayList<String>();
		lithKey = new ArrayList<String>();
		lithSymMap = new LinkedHashMap<Character, String>();
		litCFG = new FileResource(wellFile + "/litcfg.txt"); 
		lithDataMap = new LinkedHashMap<Float, String>();
		lithDepth = new ArrayList<Float>();
		lithSym = new ArrayList<Character>();  //This list holds the symbol letters from litcfg.txt
		percentages = new ArrayList<String>();
	}

	public void init() {
		
		lithLines = fillArray(lith);
		lithKey = fillArray(litCFG);
		lithSymMap = fillHashMap(lithKey);
		lithDataMap = getLithData(lithLines);
		getLithSyms();
		fillPercents();
		
		//System.out.println(lithKey);
		
		//for(String s : percentages) System.out.println(s);
		
		// Parse depth from getPercents
		
		
		StringBuilder csvHeader = new StringBuilder();
		
		// Create header line for csv file.
		
		csvHeader.append("Depth");
		for(int i = 0; i < lithSym.size(); i++){
			csvHeader.append(",");
			csvHeader.append(lithSym.get(i));
		}
		System.out.println(csvHeader);		
				
		
		int tab= 0;
		
		
	
		

		StringBuilder sb = new StringBuilder();
		int index = 0;
		int percentIndex = 0;   //This is the index of the % sign.
		String sym = null;
		String line = null;
		String element = null;
		String percent = null;
		String percentData = null;
		Float depth = null;
		//     ***Start outer for loop here***
		for(int j = 0; j < lithSym.size(); j++) {
			line = percentages.get(0).toString();
			sym = lithSym.get(j).toString(); 
			
			String p = getPercents(1);	
			for(int i = 0; i < percentages.get(1).length(); i++) {
				String e = line.substring(i, i + 1);
				tab = p.indexOf("\t");
				depth = Float.parseFloat((line.substring(0, tab)));
				//percentData = p.substring(tab).trim();
			}
			sb.append(depth);
			
			for(int i = 0; i < percentages.get(j).length(); i ++){
				element = line.substring(i,  i + 1);
				if(line.contains(sym)) {
					index = line.indexOf(sym) + 2; //Add 2 to parse over the = sign to get the digits represented the %
					percentIndex = line.indexOf("%");
					percent = line.substring(index, percentIndex);
					
				} else {
					percent = ",";
					//System.out.println("line does not contain sym " + sym);
					
				}
				
				sb.append("," + percent);
			}
			
		//	***End outer loop here***
		}
		
		System.out.println("line: " + line);
		System.out.println("sym = " + sym);
		System.out.println("index of sym = " + index);
		System.out.println("index of % = " + percentIndex);
		System.out.println("element = " + element);
		System.out.println("percent = " + percent);
		System.out.println("depth = " + depth);
		System.out.println("percentData: " + percentData);	
		System.out.println("sb = " + sb);	
		
		String fileName = "lithPercents.csv";		
		String fileFolder = "C:\\Users\\Eric\\OneDrive\\data\\";
		String fileLocation = fileFolder + fileName;
		
		
		// PrintWriter.println() stores String in RAM. Must use flush() or close() to 
		// finish writing it to the file.
		try {
			
			PrintWriter out = new PrintWriter(fileLocation);  
			//System.out.println(depth);
			//System.out.println( percentData);
			out.println(csvHeader);
			out.close();		// Flushes data from RAM to the file.
			
			
		} catch (FileNotFoundException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		
	}
	
	public void fillPercents() {
		for (int i = 0; i < lithDepth.size(); i++){
			//System.out.print(lithDepth.get(i));
			percentages.add(getPercents(i));
		}
	}
	
	public String getPercents(int depthIndex) {
		StringBuilder line = new StringBuilder();
		StringBuilder sb = new StringBuilder();
		String lithLine;
		lithLine = lithDataMap.get(lithDepth.get(depthIndex));      
		lithLine = lithLine.trim(); 						// Removes whitespace.
		lithLine = lithLine.toLowerCase();
		String[] parts = lithLine.split(",");        		//put lith in array using commas as a delimiter.
		int[] lithPercent = new int[parts.length];
		ArrayList<Integer> symbolCount = new ArrayList<Integer>();
		int counter;
		String semiCol; 
		ArrayList<String> half = new ArrayList<String>();
		
		// Use StringBuilder to put lith back into string with commas removed.
		for(String s : parts) line.append(s);   
		
		//look for the semicolon pairs so that they can be calculated as 10% instead of 20%.
		for (int t = 0; t < line.length(); t++) {
			String element = line.substring(t, (t + 1));
			if (element.equals(";")) {
				semiCol = line.substring(t-1, t+2); 
				half.add(semiCol);					// Add the semicolon pair to ArrayList half
				line.delete((t-1), (t+2));			// and delete it from StringBuilder line.
				t = 0;								// Set to zero so the loop searches for ant remaining semicolons
			}
		}
		
		//System.out.print("\t" + line.toString().toUpperCase());
		//System.out.println("\tNumber of semicolons removed = " + half.size());
		
		for(int i = 0; i < lithSym.size(); i++) {
			counter = 0;
			String s = lithSym.get(i).toString().toLowerCase();
		
			for(int k = 0; k < line.length(); k++) {
				String letter = line.substring(k, (k + 1));
				if(letter.equals(s)) counter = counter + 20;	
			}
			
			// Parse the semicolon pairs and add 10% where necessary.
			for(int k = 0; k < half.size(); k++){
				for(int l = 0; l < half.get(k).length(); l++) {
					String letter2 = half.get(k).substring(l, (l + 1));
					if(letter2.equals(s)) counter = counter + 10;
				}
			}
			
			symbolCount.add(i, counter);
			int j = 0;
			
			if(symbolCount.get(i) != 0){
				lithPercent[j] = symbolCount.get(i);
				String percentLine;
				percentLine = lithSym.get(i) + "=" + lithPercent[j] + "%\t";
				j++;
				sb.append(percentLine);
			}
		}
		// Insert the depth to the beginning of the string and chop off the decimal.
		sb.insert(0, Math.round((lithDepth.get(depthIndex))) + "\t");
		return sb.toString();
	}
	
	public void getLithSyms(){
		for (String s : lithKey) {
			lithSym.add(s.charAt(0));
		}
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
			
			delim = al.get(i).indexOf((char)32);      // 32 is ascii for " " (space).
			liths = al.get(i).substring(delim);
			
			if (!isNumeric(al.get(i).substring(0, delim))){ //Skips lines that doesn't start with a number
				header = al.get(i).substring(0);  
				hm.put(999999f, header);          	//assumes line that doesn't begin with a number is the header title.
				lithHeader = header;				// and stores it in header variable
				hm.remove(999999f);
				continue;                          
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
			hm.put(c, s);  // hm is lithSymMap
		}
		
		return hm;
	}
	
	 // Method found on Stack Overflow. Checks to see if a String is a number.
	 public static boolean isNumeric(String str) {
	    try {
	      double d = Double.parseDouble(str);
	    } catch (NumberFormatException nfe) {
	      return false;
	    } 
	    return true;
	  }
	 	
	private void out(String s){ //Shortcut method for printing to the console
		System.out.println(s);		
	}
	
	private void out(int i){ //Shortcut method for printing to the console
		System.out.println(i);		
	}
	
	public static void main(String[] args) {
		File lithtxt = new File("C:/Users/Eric/OneDrive/data/lith.txt");
		ReadLith test = new ReadLith(lithtxt);
		test.init();
		
		
		
		
	}

}

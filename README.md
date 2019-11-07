# Database
SQL-Spell Check.

	
    import java.io.BufferedReader;
    import java.io.FileReader;
    import java.io.IOException;
    import java.sql.Connection;
    import java.sql.DriverManager;
    import java.sql.PreparedStatement;
    import java.util.ArrayList;
    import java.util.Collections;
    import java.util.Comparator;
    import java.util.HashMap;
    import java.util.List;
    import java.util.Map.Entry;
    import java.util.Set;
	
    public class MySQL {
	
	public static void main(String[] args) throws Exception
	{   
		creatTable();
		
		// created HashMap for counting words & values
	    HashMap<String, Integer> wordCountMap = new HashMap<String, Integer>();
	    BufferedReader reader = null;
	     
	    try
	    {
	    	// created BufferedReader 
	        reader = new BufferedReader(new FileReader("C:\\Users\\Student\\Desktop\\Shakespear.txt."));
	         
	      //Reading the first line into currentLine
	        String currentLine = reader.readLine();
	        while (currentLine != null)
	        {   
 	            //splitting the currentLine into words  
	            String[] words = currentLine.toLowerCase().split(" ");
	            
	            //Iterating each word
	            for (String word : words)
	            {
	                //If word is already in wordCountMap, updated count
	                if(wordCountMap.containsKey(word))
	                {
	                	wordCountMap.put(word, wordCountMap.get(word)+1);
	                }
	                //otherwise inserting the word as key and 1 as its value
	                else
	                {
	                    wordCountMap.put(word, 1);
	                }
	            }
	            //Reading next line into currentLine
	            currentLine = reader.readLine();
	        }
	      //Getting entries of wordCountMapn into form of set
	        Set<Entry<String, Integer>> entrySet = wordCountMap.entrySet();
	        
	      //Creating a List passing the entrySet
	        List<Entry<String, Integer>> list = new ArrayList<Entry<String,Integer>>(entrySet);
	        
	      //Sorting the list in decreasing order of values 
	        Collections.sort(list, new Comparator<Entry<String, Integer>>() 
	        {
	            @Override
	            public int compare(Entry<String, Integer> e1, Entry<String, Integer> e2) 
	            {
	                return (e2.getValue().compareTo(e1.getValue()));
	            }
	        });

	        //Printing the repeated words in input file along with their occurrences 
	        System.out.println("Repeated Words In Input File Are :");
	        for (Entry<String, Integer> entry : list) 
	        {
	        	if (entry.getValue() > 1)
	            {
	        		System.out.println(entry.getKey() + " : "+ entry.getValue());
	            }
	        }
	    } 
	    catch (IOException e) 
	    {
	        e.printStackTrace();
	    }
	    finally
	    {
	        try
	        {
	        	//Closing the reader
	            reader.close();           
	        }
	        catch (IOException e) 
	        {
	            e.printStackTrace();
	        }
	    }
	}
	
	public static void creatTable() throws Exception {
		try {
			Connection con = getConnection();
			PreparedStatement create = con.prepareStatement("Creat Table if not exisit Words(id int NOT NULL AUTO_INCREMENT, first  varchar(255), last varchar(255), PRIMARY KEY(id))");
			create.executeUpdate();
			
		}catch(Exception e) {
			System.out.println(e);
		} finally {
			System.out.println("Function complete.");
		}
	}
	
	public static Connection getConnection() throws Exception {
		try {
			String driver = "com.mysql.cj.jdbc.Driver";
			String url = "jdbc:mysql://localhost:3306/DataBase";
			String username = "CloneWars";
			String password = "Order66";
			Class.forName(driver);

			Connection conn = DriverManager.getConnection(url, username, password);
			System.out.println("Connected");
			return conn;
		} catch (Exception e) {
			System.out.println(e);
		}

		return null;
	}
}




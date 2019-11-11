# Database
SQL-Spell Check.

	
    package dataBase;

	import java.io.*;
	import java.sql.Connection;
	import java.sql.DatabaseMetaData;
	import java.sql.DriverManager;
	import java.sql.PreparedStatement;
	import java.sql.ResultSet;
	import java.sql.SQLException;
	import java.sql.Statement;
	import java.util.*;
	import java.util.regex.*;

	class Spelling {

	public static final String DRIVER = "com.mysql.cj.jdbc.Driver";
	public static final String URL = "jdbc:mysql://localhost:3306/DataBase";
	public static final String USERNAME = "contacts";
	public static final String PASSWORD = "contacts";

	private HashMap<String, Integer> nWords;

	public Spelling(String file) throws IOException {
		nWords = new HashMap<String, Integer>();
		BufferedReader in = new BufferedReader(new FileReader(file));

		// This pattern matches any word character (letters or digits)
		Pattern p = Pattern.compile("\\w+");
		for (String temp = ""; temp != null; temp = in.readLine()) {
			Matcher m = p.matcher(temp.toLowerCase());

			// find looks for next match for pattern p (in this case a word). True if found.
			// group then returns the last thing matched.
			// The ? is a conditional expression.
			while (m.find())
				nWords.put((temp = m.group()), nWords.containsKey(temp) ? nWords.get(temp) + 1 : 1);
		}
		in.close();
	}

	private ArrayList<String> edits(String word) {
		ArrayList<String> result = new ArrayList<String>();

		// All deletes of a single letter
		for (int i = 0; i < word.length(); ++i)
			result.add(word.substring(0, i) + word.substring(i + 1));

		// All swaps of adjacent letters
		for (int i = 0; i < word.length() - 1; ++i)
			result.add(word.substring(0, i) + word.substring(i + 1, i + 2) + word.substring(i, i + 1)
					+ word.substring(i + 2));

		// All replacements of a letter
		for (int i = 0; i < word.length(); ++i)
			for (char c = 'a'; c <= 'z'; ++c)
				result.add(word.substring(0, i) + String.valueOf(c) + word.substring(i + 1));

		// All insertions of a letter
		for (int i = 0; i <= word.length(); ++i)
			for (char c = 'a'; c <= 'z'; ++c)
				result.add(word.substring(0, i) + String.valueOf(c) + word.substring(i));

		return result;
	}

	public String correct(String word) {
		// If in the dictionary, return it as correctly spelled
		if (nWords.containsKey(word))
			return word;

		ArrayList<String> list = edits(word); // Everything edit distance 1 from word
		HashMap<Integer, String> candidates = new HashMap<Integer, String>();

		// Find all things edit distance 1 that are in the dictionary. Also remember
		// their frequency count from nWords.

		for (String s : list)
			if (nWords.containsKey(s))
				candidates.put(nWords.get(s), s);

		// If found something edit distance 1 return the most frequent word
		if (candidates.size() > 0)
			return candidates.get(Collections.max(candidates.keySet()));

		// Find all things edit distance 1 from everything of edit distance 1. These
		// will be all things of edit distance 2 (plus original word).
		for (String s : list)
			for (String w : edits(s))
				if (nWords.containsKey(w))
					candidates.put(nWords.get(w), w);

		// If found something edit distance 2 return the most frequent word.
		// If not return the word with a "?" prepended. (Original just returned the
		// word.)
		return candidates.size() > 0 ? candidates.get(Collections.max(candidates.keySet())) : "?" + word;
	}

	/*
	 * public static void main(String args[]) throws IOException { if(args.length >
	 * 0) System.out.println((new Spelling("src/Dictionary.txt")).correct(args[0]));
	 * }
	 */

	public static void main(String args[]) throws Exception {

		createTable();
		post();
		get();
		Spelling corrector = new Spelling("src/Dictionary.txt");
		Scanner input = new Scanner(System.in);

		System.out.println("Enter words to correct:");
		String word = input.next();

		while (true) {
			System.out.println(word + " is corrected to " + corrector.correct(word));
			word = input.next();
		}

	}

	public static ArrayList<String> get() throws Exception {
		try {
			Connection con = getConnection();
			PreparedStatement statement = con.prepareStatement("SELECT * FROM iamalive ORDER BY first DESC LIMIT 1");

			ResultSet result = statement.executeQuery();

			ArrayList<String> array = new ArrayList<String>();
			while (result.next()) {
				System.out.print(result.getString("first"));
				System.out.print(" ");
				System.out.println(result.getString("last"));

				array.add(result.getString("last"));
			}
			System.out.println("All records have been selected!");
			return array;

		} catch (Exception e) {
			System.out.println(e);
		}
		return null;

	}

	public static void post() throws Exception {
		final String var1 = "Luc";
		final String var2 = "Nguyen";
		try {
			Connection con = getConnection();
			PreparedStatement posted = con
					.prepareStatement("INSERT INTO iamalive (first, last) VALUES ('" + var1 + "', '" + var2 + "')");

			posted.executeUpdate();
		} catch (Exception e) {
			System.out.println(e);
		} finally {
			System.out.println("Insert Completed.");
		}
	}

	public static void createTable() throws Exception {
		try {
			Connection con = getConnection();
			PreparedStatement create = con.prepareStatement(
					"Create Table if not exisit Words(id int NOT NULL AUTO_INCREMENT, first  varchar(255), last varchar(255), PRIMARY KEY(id))");
			create.executeUpdate();

		} catch (Exception e) {
			System.out.println(e);
		} finally {
			System.out.println("Function complete.");
		}
	}

	public static Connection getConnection() throws Exception {
		try {
			String driver = "com.mysql.cj.jdbc.Driver";
			String url = "jdbc:mysql://209.18.47.62:3306/DataBase";
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

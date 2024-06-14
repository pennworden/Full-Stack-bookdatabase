// These are the Serverlets for my project. These servlet files essentially handle web requests when a user inputs something online.
//This is the AddServlet file that allows users to add books
package controllers;

import java.io.IOException;

import javax.servlet.RequestDispatcher;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import dbhelpers.BookDBHelper;
import model.Book;

/**
 * Servlet implementation class AddServlet
 */
@WebServlet("/addBook")
public class AddServlet extends HttpServlet {
	private static final long serialVersionUID = 1L;
       
    /**
     * @see HttpServlet#HttpServlet()
     */
    public AddServlet() {
        super();
    }

	/**
	 * @see HttpServlet#doGet(HttpServletRequest request, HttpServletResponse response)
	 */
	protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		doPost(request, response);
	}

	/**
	 * @see HttpServlet#doPost(HttpServletRequest request, HttpServletResponse response)
	 */
	protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        // get the data
        String title = request.getParameter("title");
        String author = request.getParameter("author");
        int pages = Integer.parseInt(request.getParameter("pages"));
    
        // set up a book object
        Book book = new Book();
        book.setTitle(title);
        book.setAuthor(author);
        book.setPages(pages);
        
        // set up an dbHelper object
        BookDBHelper bdb = new BookDBHelper();
        
        // pass the book to addQuery to add to the database
        bdb.doAdd(book);
        
        // pass execution control to the ReadServlet
        String url = "/read";
        
        RequestDispatcher dispatcher = request.getRequestDispatcher(url);
        dispatcher.forward(request, response);
    }
}
// This is the DeleteServlet file that allows users to delete books
package controllers;

import java.io.IOException;

import javax.servlet.RequestDispatcher;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import dbhelpers.BookDBHelper;

/**
 * Servlet implementation class DeleteServlet
 */
@WebServlet(description = "Deletes a record for a particular bookID", urlPatterns = { "/delete" })
public class DeleteServlet extends HttpServlet {
    private static final long serialVersionUID = 1L;
       
    /**
     * @see HttpServlet#HttpServlet()
     */
    public DeleteServlet() {
        super();
        // TODO Auto-generated constructor stub
    }

    /**
     * @see HttpServlet#doGet(HttpServletRequest request, HttpServletResponse response)
     */
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        // NEVER make database changes via a GET request.
        // You don't want a web crawler accidentally deleting your data!
        throw new RuntimeException();
    }

    /**
     * @see HttpServlet#doPost(HttpServletRequest request, HttpServletResponse response)
     */
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        // get the bookID
        int bookID = Integer.parseInt(request.getParameter("bookID"));
        
        // create a dbHelper object
        BookDBHelper bdb = new BookDBHelper();
        
        // use deleteQuery to delete the record
        bdb.doDelete(bookID);
        
        // Consider: We may wish to redirect to the "/read" page rather than forward.
        // What's the difference? A redirect creates a new HTTP request, and may be 
        // used to when reloading a URL is undesirable. (Do we want a user to be able to 
        // reload a delete page?)
        //
        // How would redirect code be different?
        //
        // See, e.g., http://stackoverflow.com/questions/2047122/requestdispatcher-interface-vs-sendredirect

        // pass execution on to the ReadServlet
        String url = "/read";
        
        RequestDispatcher dispatcher = request.getRequestDispatcher(url);
        dispatcher.forward(request, response);
        
    }

//This is the servlet to read the books table
package controllers;

import java.io.IOException;
import java.sql.ResultSet;

import javax.servlet.RequestDispatcher;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import dbhelpers.BookDBHelper;

/**
 * Servlet implementation class ReadServlet
 */
@WebServlet(
        description = "Controller for reading the books table", 
        urlPatterns = { 
                "/index.jsp", 
                "/read"
        })
public class ReadServlet extends HttpServlet {
    private static final long serialVersionUID = 1L;
       
    /**
     * @see HttpServlet#HttpServlet()
     */
    public ReadServlet() {
        super();
    }

    /**
     * @see HttpServlet#doGet(HttpServletRequest request, HttpServletResponse response)
     */
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        doPost(request, response);
    }

    /**
     * @see HttpServlet#doPost(HttpServletRequest request, HttpServletResponse response)
     */
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        
        BookDBHelper bdb = new BookDBHelper();

        String author = request.getParameter("author");
        ResultSet results;
        if(author != null && !author.isEmpty()) {
            results = bdb.doReadByAuthor(author);  
        } else {
            results = bdb.doReadAll();
        }

        
        String table = bdb.getHTMLTable(results);

        
        request.setAttribute("table", table);
        String url = "/read.jsp";

        RequestDispatcher dispatcher = request.getRequestDispatcher(url);
        dispatcher.forward(request, response);
    }

}
// Servlet to handle the processing book updates
package controllers;

import java.io.IOException;

import javax.servlet.RequestDispatcher;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import dbhelpers.BookDBHelper;
import model.Book;

/**
 * Servlet implementation class UpdateFormServlet
 */
@WebServlet("/update")
public class UpdateFormServlet extends HttpServlet {
	private static final long serialVersionUID = 1L;
       
    /**
     * @see HttpServlet#HttpServlet()
     */
    public UpdateFormServlet() {
        super();
    }

	
	/**
	 * @see HttpServlet#doPost(HttpServletRequest request, HttpServletResponse response)
	 */
	protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		 // get the bookID
        int bookId = Integer.parseInt(request.getParameter("bookID"));
        
        // create dbHelper class
        BookDBHelper bdb = new BookDBHelper();
        
        // Use method to get the book data
        Book book = bdb.doReadOne(bookId);
            
        // pass Book and control to the updateForm.jsp
        request.setAttribute("book", book);
        
        String url = "/updateForm.jsp";
        
        RequestDispatcher dispatcher = request.getRequestDispatcher(url);
        dispatcher.forward(request, response);
    }

}
//Servlet to update book information
package controllers;

import java.io.IOException;

import javax.servlet.RequestDispatcher;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import dbhelpers.BookDBHelper;
import model.Book;

/**
 * Servlet implementation class UpdateServlet
 */
@WebServlet("/updateBook")
public class UpdateServlet extends HttpServlet {
	private static final long serialVersionUID = 1L;
       
    /**
     * @see HttpServlet#HttpServlet()
     */
    public UpdateServlet() {
        super();
    }


	/**
	 * @see HttpServlet#doPost(HttpServletRequest request, HttpServletResponse response)
	 */
	protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		// get the form data and set up a Book object
        int bookID = Integer.parseInt(request.getParameter("bookID"));
        String title = request.getParameter("title");
        String author = request.getParameter("author");
        int pages = Integer.parseInt(request.getParameter("pages"));

        // create a book object with values
        Book book = new Book();
        book.setBookID(bookID);
        book.setAuthor(author);
        book.setTitle(title);
        book.setPages(pages);

        // create an dbHelper object and use it to update the book
        BookDBHelper bdb = new BookDBHelper();
        bdb.doUpdate(book);

        // pass control on to the ReadServlet
        String url = "/read";

        RequestDispatcher dispatcher = request.getRequestDispatcher(url);
        dispatcher.forward(request, response);
	}

}



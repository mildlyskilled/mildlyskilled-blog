---
title: "MySQL and Scala - Simple selects"
date: 2011-07-29T14:15:42+01:00
draft: false
author: "Kwabena Aning"
tags: ["scala", "mysql", "linux"]
type: "post"
---

So after about an hour of research and experimenting, I found out how to make Scala work with MySQL without too much hassle, I found a really helpful guide [here](https://github.com/ollekullberg/SimpleOrder). Going down the SBT route, I first created a project and all that, then inside of the build folder I created a Scala class that ensured that when I started building, I would have the right dependencies downloaded and ready.


    import sbt._

    class MyDataEngineProject(info: ProjectInfo)
        extends DefaultProject(info) {
      // Declare MySQL connector Dependency
      val mysql = "mysql" % "mysql-connector-java" % "5.1.17"
    }


Once you have got this in project/build/MyDataEngineProject.scala, you are ready to start the connection stuff. In src/main/scala you can start creating your application here. I use the "def main" approach so when i start from the command line i simply pass in my parameters and I'm ready to go:


    import java.sql.{Connection, DriverManager, SQLException, ResultSet}

    object DataEngine {

    	private var driverLoaded = false

    	private def loadDriver()  {
    		try{
    			Class.forName("com.mysql.jdbc.Driver").newInstance
    			driverLoaded = true
    		}catch{
    			case e: Exception  => {
    			    println("ERROR: Driver not available: " + e.getMessage)
    			    throw e
    			}
    		}
    	}

    	def main(args: Array[String]) {
    		if (args.length < 1){
    			println("No arguments provided, exitting...")
    			return
    		}
    		println ("Attempting to load MySQL JBDC Driver...")
    		println
    		this.synchronized {
    		  if(! driverLoaded) loadDriver()
    		}

    		val conString = "jdbc:mysql://localhost:3306/db_name?user="+args(0)+"&password="+args(1)
    		classOf[com.mysql.jdbc.Driver]
    		val con = DriverManager.getConnection(conString)
    		try{
    			val handle = con.createStatement(ResultSet.TYPE_FORWARD_ONLY, ResultSet.CONCUR_READ_ONLY)
    			val rs = handle.executeQuery("select * from auth limit 1")
    			println ("Connection established...")
    			while(rs.next){
    				println("User Handle: "+rs.getString("handle"))
    			}
    		}catch{
    			case e: Exception  => {
    		    	println("ERROR: No connection: " + e.getMessage)
    		    	throw e
    		  	}
    			case _ => println("A problem!")
    		}finally{
    			println("Closing connection and exiting...")
    			con.close()
    		}
      	}
    }


Save the file as Main.scala and you can just type in run with your db username and password as space-separated arguments and you're ready to go. Of course you will need to change some bits in here to reflect your database schema, but aside that it's that simple. In my next post I will work on inserting data and updating data as well.

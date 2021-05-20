

# create a dao package , and create these file under it : 
`DAO.java : `

```java
package dao;

import java.sql.Connection;
public abstract class DAO<T> {
protected Connection connection = null;
public DAO(Connection conn){
this.connection = conn;
}
public abstract boolean create(T entity);
public abstract boolean delete(T entity);
public abstract boolean update(T entity);
public abstract T find(String id);
}
```
`DAOException.java :`

```java
package dao;

public class DAOException extends Exception {
  /**
   * Code de l'erreur.
   * -1 s'il y a une exception chaînée ;
   * 1 pour connexion pas ouverte ;
   * 2 pour connexion déjà ouverte ;
   * 4 si pas de transaction en cours ;
   * ...
   */
  private int codeErreur;

  public DAOException() {
  }

  public DAOException(String message) {
    this(message, 0);
  }

  /**
   * Crée une nouvelle exception avec un message et une cause donnés.
   * @param message le message qui explique le problème.
   * @param cause une exception qui est la cause du problème. Le type de cette
   * exception doit être caché à l'utilisateur du DAO et ne pas apparaître
   * dans l'interface de la classe DaoException. Cette cause peut être
   * connue de l'utilisateur par l'appel de la méthode getCause() héritée
   * de Exception.
   * @param codeErreur code erreur du support de persistance.
   */
  public DAOException(String message, Throwable cause, int codeErreur) {
    super(message, cause);
    this.codeErreur = codeErreur;
  }
  
  public DAOException(String message, Throwable cause) {
    this(message, cause, -1);
  }

  public DAOException(Throwable cause) {
    this("Erreur liée aux DAO", cause, -1);
  }
  
  public DAOException(String message, int codeErreur) {
    this(message, null, codeErreur);
  }
  
  public int getCode() {
    return codeErreur;
  }
}
```
`DapartementDao.java :`
```java
package dao;

public class DAOException extends Exception {
  /**
   * Code de l'erreur.
   * -1 s'il y a une exception chaînée ;
   * 1 pour connexion pas ouverte ;
   * 2 pour connexion déjà ouverte ;
   * 4 si pas de transaction en cours ;
   * ...
   */
  private int codeErreur;

  public DAOException() {
  }

  public DAOException(String message) {
    this(message, 0);
  }

  /**
   * Crée une nouvelle exception avec un message et une cause donnés.
   * @param message le message qui explique le problème.
   * @param cause une exception qui est la cause du problème. Le type de cette
   * exception doit être caché à l'utilisateur du DAO et ne pas apparaître
   * dans l'interface de la classe DaoException. Cette cause peut être
   * connue de l'utilisateur par l'appel de la méthode getCause() héritée
   * de Exception.
   * @param codeErreur code erreur du support de persistance.
   */
  public DAOException(String message, Throwable cause, int codeErreur) {
    super(message, cause);
    this.codeErreur = codeErreur;
  }
  
  public DAOException(String message, Throwable cause) {
    this(message, cause, -1);
  }

  public DAOException(Throwable cause) {
    this("Erreur liée aux DAO", cause, -1);
  }
  
  public DAOException(String message, int codeErreur) {
    this(message, null, codeErreur);
  }
  
  public int getCode() {
    return codeErreur;
  }
}
```
 `EtudiantDAO.java :`
 
 ```java
 package dao;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import entity.Etudiant;
import entity.Departement;
public class EtudiantDAO extends DAO <Etudiant> {
public EtudiantDAO (Connection conn){
super(conn);
}
public boolean create (Etudiant entity){
boolean ok = true;
try{
String reqInsert = "insert into Etudiant (Matricule, nom,Adresse,Age, NDept) values (?,?,?,?,?)";
PreparedStatement pstmt = this.connection.prepareStatement(reqInsert);
pstmt. setString (1, entity.getMatricule());
pstmt.setString(2, entity.getNom());
pstmt. setString (3, entity.getAdresee());
pstmt.setInt(4, entity.getAge());
pstmt. setString (5, entity.getNDept());
pstmt.executeUpdate();
}
catch(SQLException e){
ok = false;
e.printStackTrace();
}
return ok;
}
public Etudiant find(String id){
Etudiant e = new Etudiant ();
try{
String select = "select * from Etudiant where Matricule = ?";
PreparedStatement pstmt;
pstmt = this.connection.prepareStatement(select, ResultSet.TYPE_SCROLL_SENSITIVE);
pstmt. setString (1,id);
ResultSet rs = pstmt.executeQuery();
if (rs.next()){
e.setMatricule(id); 
e.setNom(rs.getString(2));
//Ici nous devons créer l'objet Departement dont le numéro figure comme clé //étrangére dans le tuple récupéré. Pour cela nous utilisons DepartementDao.
//L'objet Departement créé est ensuite associé à l'objet Etudiant retourné
DepartementDao deptDao = new DepartementDao (this.connection);
e.setNDept(deptDao.find(rs. getString ("NDept")).getNDept());
}
}
catch(SQLException e1){
e1.printStackTrace();
}
return e;
}
public boolean delete(Etudiant entity){ return false;}
public boolean update(Etudiant entity){ return false;}
}
```
# create an entity package , and create these file under it : 

`Departement.java :`

```java
package entity;

import java.util.*;

public class Departement {
	private String NDept,Nomdept;
	private Set<Etudiant> etudinats = new HashSet< Etudiant >();
	 public Departement(){};
	 public String getNDept(){return NDept;}
	 public String getNomdept(){return Nomdept;}
	 
	 public void setNDept(String NDept){this.NDept= NDept;}
	 public void setNomdept(String Nomdept){this.Nomdept= Nomdept;}
	 
	 public void addEtudiant(Etudiant e){
		 if (e.getNDept()==null) etudinats.add(e);}
	 public void removeEtudiant(Etudiant e){etudinats.remove(e);}
	 


}
```
`Etudiant.java :`

```java
package entity;

public class Etudiant {
	private String Matricule, Nom , Adresee ,NDept;
	private int Age;
	 public Etudiant(){};
	 public Etudiant(String Matricule, String Nom , String Adresee ,int Age,String NDept){
		 this.Matricule=Matricule;
		 this.Nom=Nom;
		 this.Adresee=Adresee;
		 this.Age=Age;
		 this.NDept=NDept;
		 
	 }
	 public String getMatricule(){return Matricule;}
	 public String getNom(){return Nom;}
	 public String getAdresee(){return Adresee;}
	 public String getNDept(){return NDept;}
	 public int getAge(){return Age;}
	 public void setMatricule(String Matricule){this.Matricule= Matricule;}
	 public void setNom(String Nom){this.Nom= Nom;}
	 public void setAdresee(String Adresee){this.Adresee= Adresee;}
	 public void setNDept(String NDept){this.NDept= NDept;}
	 public void setAge(int Age){this.Age= Age;}

}
```

# create a metier package , and create these file under it : 
 `Test.java :`
 
 ```java
 package entity;

public class Etudiant {
	private String Matricule, Nom , Adresee ,NDept;
	private int Age;
	 public Etudiant(){};
	 public Etudiant(String Matricule, String Nom , String Adresee ,int Age,String NDept){
		 this.Matricule=Matricule;
		 this.Nom=Nom;
		 this.Adresee=Adresee;
		 this.Age=Age;
		 this.NDept=NDept;
		 
	 }
	 public String getMatricule(){return Matricule;}
	 public String getNom(){return Nom;}
	 public String getAdresee(){return Adresee;}
	 public String getNDept(){return NDept;}
	 public int getAge(){return Age;}
	 public void setMatricule(String Matricule){this.Matricule= Matricule;}
	 public void setNom(String Nom){this.Nom= Nom;}
	 public void setAdresee(String Adresee){this.Adresee= Adresee;}
	 public void setNDept(String NDept){this.NDept= NDept;}
	 public void setAge(int Age){this.Age= Age;}

}
```
# create a utils package , and create these file under it : 
`MySqlConnection.java :`

```java
package Utils;
	import java.sql.Connection;
	import java.sql.DriverManager;
	import java.sql.SQLException;
	public class MySqlConnection {
		private static String url = "jdbc:mysql://localhost/Etudiant";
		private static String user = "root";
		private static String password = "";
		private static Connection cn;
		private static String name = "localhost/Etudiant";
		private MySqlConnection (){
		      try {
		           Class.forName("com.mysql.jdbc.Driver");
		           cn = DriverManager.getConnection(url, user, password);
		           } 
		      catch (ClassNotFoundException ex) {
		    	   System.out.println("Impossible de charger le driver " +ex.getMessage());
			        }
		      catch (SQLException ex) {
			        	System.out.println("Erreur de connexion\n" +ex.getMessage());
			            
			        }
			  
		}
		public static Connection getInstanc(){
			if(cn==null){
				new MySqlConnection();
				System.out.println("Instanciation de la connection à  "+ name);
			}
			else 
				System.out.println("Connection à  "+ name + " existante");
			return cn;
			}
		public static String getName() {return name;}
			
		}
```




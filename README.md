In this tutorial, I am going to show you how you can create a simple CRUD in PHP with search and pagination functionality. For front end or user interface, I use Bootstrap 4 you can use any design framework as per your need.

Simple CRUD link without pagination is listed below.

PHP CRUD in Bootstrap 4 with search functionality
PHP CRUD in Bootstrap 3 with a search functionality
In this example, I will use the POD base DB class that holds all queries with a secure connection. To download DB class click here. And also download PHP pagination class from here. Add PHP pagination class file into the config.php with the help of include_once() function.

How to set up the config file.
Create a file with the name of config.php and then open the config.php file and paste below code in it.

Remember include config file in all PHP files with the help of include_once() function. It is the main file that holds a database connection. So without DB connection, you can not perform any DB operation into a database.


<?php
  include_once('include/Database.php');
  include_once('include/paginator.class.php');
  define('SS_DB_NAME', 'test');
  define('SS_DB_USER', 'root');
  define('SS_DB_PASSWORD', '');
  define('SS_DB_HOST', 'localhost');
  
  $dsn  =   "mysql:dbname=".SS_DB_NAME.";host=".SS_DB_HOST."";
  $pdo  =   "";
  try {
    $pdo = new PDO($dsn, SS_DB_USER, SS_DB_PASSWORD);
  }catch (PDOException $e) {
    echo "Connection failed: " . $e->getMessage();
  }
  $db    =   new Database($pdo);
  $pages =   new Paginator();
?>
In the config file, I already added the Database class with the help of include_once function. So you do not need to add this file again and again.

Paste below query to create table in mysql.


CREATE TABLE `users` (
   `id` INT NOT NULL AUTO_INCREMENT ,
   `username` VARCHAR(64) NOT NULL ,
   `useremail` VARCHAR(128) NOT NULL ,
   `userphone` VARCHAR(24) NOT NULL ,
   `dt` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP() ,
   PRIMARY KEY (`id`)
) ENGINE = MyISAM;
Create the add-user.php file and paste below code just after the body tag.


<form method="post">
    <div class="form-group">
        <label>User Name <span class="text-danger">*</span></label>
        <input type="text" name="username" id="username" class="form-control" placeholder="Enter user name" required>
    </div>
    <div class="form-group">
        <label>User Email <span class="text-danger">*</span></label>
        <input type="email" name="useremail" id="useremail" class="form-control" placeholder="Enter user email" required>
    </div>
    <div class="form-group">
        <label>User Phone <span class="text-danger">*</span></label>
        <input type="tel" name="userphone" id="userphone" class="form-control" placeholder="Enter user phone" required>
    </div>
    <div class="form-group">
        <button type="submit" name="submit" value="submit" id="submit" class="btn btn-primary"><i class="fa fa-fw fa-plus-circle"></i> Add User</button>
    </div>
</form>
The above code snippet is based on Bootstrap 4. After creating a form paste below code just before the <form> tag. Below PHP code is used to take form data and submit into database.


<?php
if(isset($_REQUEST['submit']) and $_REQUEST['submit']!=""){
    extract($_REQUEST);
    if($username==""){
        header('location:'.$_SERVER['PHP_SELF'].'?msg=un');
        exit;
    }elseif($useremail==""){
        header('location:'.$_SERVER['PHP_SELF'].'?msg=ue');
        exit;
    }elseif($userphone==""){
        header('location:'.$_SERVER['PHP_SELF'].'?msg=up');
        exit;
    }else{
        $data   =   array(
                        'username'=>$username,
                        'useremail'=>$useremail,
                        'userphone'=>$userphone,
                        );
        $insert =   $db->insert('users',$data);
        if($insert){
            header('location:'.$_SERVER['PHP_SELF'].'?msg=ras');
            exit;
        }else{
            header('location:'.$_SERVER['PHP_SELF'].'?msg=rna');
            exit;
        }
    }
}
?>
Success & Error message handling is listed below. You can past below code any where in the file where you want to show the message.


<?php
if(isset($_REQUEST['msg']) and $_REQUEST['msg']=="un"){
    echo    '<div class="alert alert-danger"><i class="fa fa-exclamation-triangle"></i> User name is mandatory field!</div>';
}elseif(isset($_REQUEST['msg']) and $_REQUEST['msg']=="ue"){
    echo    '<div class="alert alert-danger"><i class="fa fa-exclamation-triangle"></i> User email is mandatory field!</div>';
}elseif(isset($_REQUEST['msg']) and $_REQUEST['msg']=="up"){
    echo    '<div class="alert alert-danger"><i class="fa fa-exclamation-triangle"></i> User phone is mandatory field!</div>';
}elseif(isset($_REQUEST['msg']) and $_REQUEST['msg']=="ras"){
    echo    '<div class="alert alert-success"><i class="fa fa-thumbs-up"></i> Record added successfully!</div>';
}elseif(isset($_REQUEST['msg']) and $_REQUEST['msg']=="rna"){
    echo    '<div class="alert alert-danger"><i class="fa fa-exclamation-triangle"></i> Record not added <strong>Please try again!</strong></div>';
}
?>
Create the browse-users.php file that shows user submitted data. So, in this file, we also need to set up pagination and search. So, first of all, we need to set up search conditions. Consider below code for search.


<?php
    $condition  =   '';
    if(isset($_REQUEST['username']) and $_REQUEST['username']!=""){
        $condition  .=  ' AND username LIKE "%'.$_REQUEST['username'].'%" ';
    }
    if(isset($_REQUEST['useremail']) and $_REQUEST['useremail']!=""){
        $condition  .=  ' AND useremail LIKE "%'.$_REQUEST['useremail'].'%" ';
    }
    if(isset($_REQUEST['userphone']) and $_REQUEST['userphone']!=""){
        $condition  .=  ' AND userphone LIKE "%'.$_REQUEST['userphone'].'%" ';
    }
     
    //Main queries
    $pages->default_ipp  =   15; //total records show on per page
    $sql    = $db->getRecFrmQry("SELECT * FROM users WHERE 1 ".$condition."");
    $pages->items_total  =   count($sql);
    $pages->mid_range    =   9;
    $pages->paginate(); 
      
    $userData   =   $db->getRecFrmQry("SELECT * FROM users WHERE 1 ".$condition." ORDER BY id DESC ".$pages->limit."");
 
?>
After above script, need to display user data in tabular format for that consider below script.


<table class="table table-striped table-bordered">
    <thead>
        <tr class="bg-primary text-white">
            <th>Sr#</th>
            <th>User Name</th>
            <th>User Email</th>
            <th>User Phone</th>
            <th class="text-center">Action</th>
        </tr>
    </thead>
    <tbody>
        <?php
        $s  =   '';
        foreach($userData as $val){
            $s++;
        ?>
        <tr>
            <td><?php echo $s;?></td>
            <td><?php echo $val['username'];?></td>
            <td><?php echo $val['useremail'];?></td>
            <td><?php echo $val['userphone'];?></td>
            <td align="center">
                <a href="edit-users.php?editId=<?php echo $val['id'];?>" class="text-primary"><i class="fa fa-fw fa-edit"></i> Edit</a> | 
                <a href="delete.php?delId=<?php echo $val['id'];?>" class="text-danger"><i class="fa fa-fw fa-trash"></i> Delete</a>
            </td>
        </tr>
        <?php } ?>
    </tbody>
</table>

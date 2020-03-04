# tars2php documentation

## Introduction
The `tars2php` module is to automatically generate client-side and server-side php code through the tars protocol file. (The server-side code is mainly frame code, and the actual business logic needs to be implemented by the user.)
## Instructions 
If you only have the requirements for packaging and unpacking, then the use process is as follows:
1. Prepare a tars protocol file, such as `example.tars`

2. Write a tars.proto.php file, which is a configuration file used to generate code.
    
    ```php
    //The servant name for this example is PHPTest.PHPServer.obj    
    return array(    
        'appName' => 'PHPTest',    
        'serverName' => 'PHPServer', 
        'objName' => 'obj',
        'withServant' => true,//whether it is server-side or client-side  
        'tarsFiles' => array(    
            './example.tars' //The path of the tars file
        ),    
        'dstPath' => './server/', //Location of the generated php file  
        'namespacePrefix' => 'Server\servant', //the namespace prefix of the php file
    );   
    ```
    
    
    
3. Execute `php ./tars2php.php ./tars.proto.php`

4. The tool will automatically generate a three-level directory structure according to the servant name. In the demo, the `PHPTest/PHPServer/obj/` directory is generated under the ./server directory. The classers in the obj directory are the php objects corresponding to the struct.
 As in struct in example.tars:
    
    ```  
        struct SimpleStruct {    
            0 require long id=0;    
            1 require unsigned int count=0;    
            2 require short page=0;    
        };    
    ```
    
    Generate classes/SimpleStruct.php
    
    ```php    
        <?php    
            
        namespace Server\servant\PHPTest\PHPServer\obj\classes;    
            
        class SimpleStruct extends \TARS_Struct {    
           const ID = 0; //tars协议中的tag    
           const COUNT = 1;    
           const PAGE = 2;    
               
           public $id; //元素的实际值    
           public $count;     
           public $page;     
               
           protected static $_fields = array(    
              self::ID => array(    
                 'name'=>'id', //tars协议中没个元素的name    
                 'required'=>true, //tars协议中是require或者optional    
                 'type'=>\TARS::INT64, //类型    
                 ),    
              self::COUNT => array(    
                 'name'=>'count',    
                 'required'=>true,    
                 'type'=>\TARS::UINT32,    
                 ),    
              self::PAGE => array(    
                 'name'=>'page',    
                 'required'=>true,    
                 'type'=>\TARS::SHORT,    
                 ),    
           );    
            
           public function __construct() {    
              parent::__construct('PHPTest_PHPServer_obj_SimpleStruct', self::$_fields);    
           }    
        }    
    ```
    
    
    
5. The interface section in `example.tars` will automatically generate a separate php file named interface. For example, `int testLofofTags(LotofTags tags, out LotofTags outtags);` The method generated by the interface is as follows

    - server part

       ```php  
          <?php  
              //Note that the comment part of the generated file will be converted to php code when the server is started. If not necessary, please do not modify it. 
             /**    
              * @param struct $tags \Server\servant\PHPTest\PHPServer\obj\classes\LotofTags    
              * @param struct $outtags \Server\servant\PHPTest\PHPServer\obj\classes\LotofTags =out=    
              * @return int     
              */    
             public function testLofofTags(LotofTags $tags,LotofTags &$outtags);    
       ```

    - client part

       ```php   
          <?php  
             try {    
                $requestPacket = new RequestPacket(); //parameters required to build the request packet    
                $requestPacket->_iVersion = $this->_iVersion;    
                $requestPacket->_funcName = __FUNCTION__;    
                $requestPacket->_servantName = $this->_servantName;
                    
                $encodeBufs = [];
              
                $__buffer = TUPAPIWrapper::putStruct("tags",1,$tags,$this->_iVersion);
                $encodeBufs['tags'] = $__buffer;
                    
                $requestPacket->_encodeBufs = $encodeBufs;
              
                $sBuffer = $this->_communicator->invoke($requestPacket,$this->_iTimeout); 
              
                $ret = TUPAPIWrapper::getStruct("outtags",2,$outtags,$sBuffer,$this->_iVersion); // Extract the first output parameter outtags from the returned package    
                return TUPAPIWrapper::getInt32("",0,$sBuffer,$this->_iVersion); //solve the return parameter return parameter name is empty, tag is 0 
             }    
             catch (\Exception $e) {    
                throw $e;    
             }    
       ```
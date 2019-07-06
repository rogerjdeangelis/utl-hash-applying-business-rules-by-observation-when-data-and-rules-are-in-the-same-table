# utl-hash-applying-business-rules-by-observation-when-data-and-rules-are-in-the-same-table
HASH: Applying business rules by observation when data and rules are in the same table 
    HASH: Applying business rules by observation when data and rules are in the same table                                                   
                                                                                                                                             
    Techinque by                                                                                                                             
                                                                                                                                             
    Bartosz Jablonski                                                                                                                        
    yabwon@gmail.com                                                                                                                         
                                                                                                                                             
    Thanks Bart et all                                                                                                                       
                                                                                                                                             
    Executing code contained in data within a datastep is potetially very usefull.                                                           
    I have not seen this novel tecnique before.                                                                                              
                                                                                                                                             
    SAS-L Posting                                                                                                                            
    https://listserv.uga.edu/cgi-bin/wa?A2=SAS-L;1dcc6dbe.1907a                                                                              
                                                                                                                                             
    *_                   _                                                                                                                   
    (_)_ __  _ __  _   _| |_                                                                                                                 
    | | '_ \| '_ \| | | | __|                                                                                                                
    | | | | | |_) | |_| | |_                                                                                                                 
    |_|_| |_| .__/ \__,_|\__|                                                                                                                
            |_|                                                                                                                              
    ;                                                                                                                                        
                                                                                                                                             
    data have;                                                                                                                               
                                                                                                                                             
     length x y 8 c $ 10 code $ 50;                                                                                                          
                                                                                                                                             
     code = '(x > 1) and c=''A''';            x= 3; y=11; c="A"; output;                                                                     
     code = '(x < 1 and (9 < y < 13))';       x=-3; y=17; c="B"; output;                                                                     
     code = '(x < 1) or C in ("C","D","E")';  x= 3; y=11; c="C"; output;                                                                     
     code = '(y < 1)';                        x=-3; y=11; c="D"; output;                                                                     
                                                                                                                                             
    run;                                                                                                                                     
                                                                                                                                             
                                                            | RULES                                                                          
                                                            | =====                                                                          
                                                            |                                                                                
    Up to 40 obs WORK.HAVE total obs=4                      | we want to get obs 1 and 3                                                     
                                                            | (since values of x, y, and c satisfy condition in "code" variable).            
                                                            |                                                                                
    Obs     X     Y    C    CODE                            |                                                                                
                                                            |                                                                                
     1      3    11    A    (x > 1) and c='A'               |  TRUE    (x > 1) and c='A'                                                     
     2     -3    17    B    (x < 1 and (9 < y < 13))        |  FALSE   y is not between 9 and 13                                             
     3      3    11    C    (x < 1) or C in ("C","D","E")   |  TRUE    c in ("C","D","E")                                                    
     4     -3    11    D    (y < 1)                         |  FALSE   y is not less that one do not use                                     
                                                            |                                                                                
    *            _               _                                                                                                           
      ___  _   _| |_ _ __  _   _| |_                                                                                                         
     / _ \| | | | __| '_ \| | | | __|                                                                                                        
    | (_) | |_| | |_| |_) | |_| | |_                                                                                                         
     \___/ \__,_|\__| .__/ \__,_|\__|                                                                                                        
                    |_|                                                                                                                      
    ;                                                                                                                                        
                                                                                                                                             
    from WANT total obs=2                                                                                                                    
                                                                                                                                             
    Obs    X     Y    C    CODE                                                                                                              
                                                                                                                                             
     1     3    11    A    (x > 1) and c='A'                                                                                                 
     3     3    11    C    (x < 1) or C in ("C","D","E")                                                                                     
                                                                                                                                             
    * GERERATED CODE;                                                                                                                        
                                                                                                                                             
    data want;                                                                                                                               
                                                                                                                                             
    set have;                                                                                                                                
                                                                                                                                             
       obs=_n_;                                                                                                                              
      select;                                                                                                                                
           when (code = "(x < 1 and (9 < y < 13))") do;                                                                                      
                     if ((x < 1 and (9 < y < 13))) then output;                                                                              
           end;                                                                                                                              
           when (code = "(x < 1) or C in ("C","D","E")") do;                                                                                 
                     if ((x < 1) or C in ("C","D","E")) then output;                                                                         
           end;                                                                                                                              
           when (code = "(x > 1) and c='A'") do;                                                                                             
                     if ((x > 1) and c='A') then output;                                                                                     
           end;                                                                                                                              
           when (code = "(y < 1)") do;                                                                                                       
                     if ((y < 1)) then output; end;                                                                                          
          otherwise;                                                                                                                         
      end;                                                                                                                                   
    run;                                                                                                                                     
                                                                                                                                             
                                                                                                                                             
    *          _       _   _                                                                                                                 
     ___  ___ | |_   _| |_(_) ___  _ __                                                                                                      
    / __|/ _ \| | | | | __| |/ _ \| '_ \                                                                                                     
    \__ \ (_) | | |_| | |_| | (_) | | | |                                                                                                    
    |___/\___/|_|\__,_|\__|_|\___/|_| |_|                                                                                                    
                                                                                                                                             
    ;                                                                                                                                        
                                                                                                                                             
    * pretty neat;                                                                                                                           
                                                                                                                                             
    options nonotes;                                                                                                                         
    data want;                                                                                                                               
      set                                                                                                                                    
        have                                                                                                                                 
      curobs=curobs indsname=indsname end=end;                                                                                               
                                                                                                                                             
      if missing(code) then                                                                                                                  
        do;                                                                                                                                  
          output;                                                                                                                            
        end;                                                                                                                                 
      else                                                                                                                                   
        do;                                                                                                                                  
          length strX $ 1024;                                                                                                                
          strX = cat(                                                                                                                        
            strip(indsname)                                                                                                                  
            , '(obs = ', curobs                                                                                                              
            , ' firstobs = ', curobs                                                                                                         
            , ' where = (', strip(code), '))'                                                                                                
            );                                                                                                                               
          declare hash H(dataset:strX);                                                                                                      
          rc = H.defineKey("code");                                                                                                          
          rc = H.defineDone();                                                                                                               
          if H.num_items > 0 then output;                                                                                                    
          rc = H.delete();                                                                                                                   
        end;                                                                                                                                 
                                                                                                                                             
      if end then put _N_=;                                                                                                                  
    run;                                                                                                                                     
    options notes;                                                                                                                           
                                                                                                                                             
                                                                                                                                             
    * __       _ _                   _                                                                                                       
     / _|_   _| | |  _ __   ___  ___| |_                                                                                                     
    | |_| | | | | | | '_ \ / _ \/ __| __|                                                                                                    
    |  _| |_| | | | | |_) | (_) \__ \ |_                                                                                                     
    |_|  \__,_|_|_| | .__/ \___/|___/\__|                                                                                                    
                    |_|                                                                                                                      
    ;                                                                                                                                        
                                                                                                                                             
    Hi SAS-Lers,                                                                                                                             
                                                                                                                                             
    I had this cool discussion at the office today and the result was very                                                                   
    interesting (at lest for me) code which I like to share.                                                                                 
                                                                                                                                             
    Idea was as old as programming: you have a dataset which contain both data and                                                           
    selection conditions (i.e. if a condition is satisfied than output an observation).                                                      
    I know that keeping both data and logic in the same place is not very "good practice" but that was the setup.                            
                                                                                                                                             
    So we have this dataset:                                                                                                                 
                                                                                                                                             
    data have;                                                                                                                               
    length x y 8 c $ 10 code $ 50;                                                                                                           
    code = '(x > 1) and c=''A''';            x= 3; y=11; c="A"; output;                                                                      
    code = '(x < 1 and (9 < y < 13))';       x=-3; y=17; c="B"; output;                                                                      
    code = '(x < 1) or C in ("C","D","E")';  x= 3; y=11; c="C"; output;                                                                      
    code = '(y < 1)';                        x=-3; y=11; c="D"; output;                                                                      
    run;                                                                                                                                     
                                                                                                                                             
    and                                                                                                                                      
                                                                                                                                             
    My first approach was:                                                                                                                   
    1) to extract unique values from variable "code",                                                                                        
    2) to use "filename t temp;" and write my "own" code                                                                                     
    something like this:                                                                                                                     
                                                                                                                                             
    proc sql;                                                                                                                                
      create table code_b as                                                                                                                 
      select distinct QUOTE(strip(code)) as q                                                                                                
      from have;                                                                                                                             
    quit;                                                                                                                                    
                                                                                                                                             
    filename want TEMP lrecl=2000;                                                                                                           
                                                                                                                                             
    data _null_;                                                                                                                             
      file have;                                                                                                                             
      set code_b end=EOF;                                                                                                                    
      length _X_ $ 2000;                                                                                                                     
                                                                                                                                             
      if _N_ = 1 then                                                                                                                        
      do;                                                                                                                                    
       put "data want;";                                                                                                                     
       put "set have;";                                                                                                                      
       put "select;";                                                                                                                        
      end;                                                                                                                                   
                                                                                                                                             
      _X_ = "when (code = "                                                                                                                  
          || strip(q)                                                                                                                        
          || ") do; if ("                                                                                                                    
          || dequote(q)                                                                                                                      
          || ") then output; end;";                                                                                                          
      put _X_;                                                                                                                               
                                                                                                                                             
      if EOF = 1 then                                                                                                                        
      do;                                                                                                                                    
       put "otherwise;";                                                                                                                     
       put "end;";                                                                                                                           
       put "run;";                                                                                                                           
      end;                                                                                                                                   
    run;                                                                                                                                     
                                                                                                                                             
    %include want / SOURCE2;                                                                                                                 
    filename want;                                                                                                                           
                                                                                                                                             
    Thing is that such approach requires to read data twice and                                                                              
    requires "writing" code. And the answer to this solution was: "I thought about                                                           
    something which will execute on the fly while reading an observation.                                                                    
    I guess you can't do it in SAS". I scratched my head and said "Ok, challenge accepted"                                                   
    (there is no such thing like "you can't do it in SAS")                                                                                   
                                                                                                                                             
    So, my first approach was dosubl() and it goes like this:                                                                                
    1) get the name of a dataset you are reading and number of read observation                                                              
    2) create string of a data step code testing "code" on this very observation                                                             
    3) return macrovariable                                                                                                                  
                                                                                                                                             
    data testc2;                                                                                                                             
      set                                                                                                                                    
        testc                                                                                                                                
      curobs=curobs indsname=indsname                                                                                                        
      ;                                                                                                                                      
      length strX $ 1024;                                                                                                                    
      strX=cat(                                                                                                                              
       ' options nonotes nosource; data _null_; '                                                                                            
      ,' set ', strip(indsname), ' (obs = ', curobs, ' firstobs = ', curobs , ');'                                                           
      ,' if (', strip(code), ') then call symputX("_TEST_",1,"G");'                                                                          
      ,' else call symputX("_TEST_",0,"G");'                                                                                                 
      ,' stop; run;'                                                                                                                         
      );                                                                                                                                     
      _rc_ = dosubl(strX);                                                                                                                   
      if symget("_TEST_") = "1" then output;                                                                                                 
                                                                                                                                             
      drop strX _rc_;                                                                                                                        
    run;                                                                                                                                     
                                                                                                                                             
    It worked but has one big flaw - since it uses dosubl() it doesn't scale (at all). For 1000 obs it worked for:                           
    real time           3:01.42                                                                                                              
    user cpu time       54.15 seconds                                                                                                        
    slow as hell...                                                                                                                          
                                                                                                                                             
    So I scratched harder on my head and than it hit me:                                                                                     
    1) hash tables allows to use variables (not only text strings) as methods arguments                                                      
    2) it can read in data from a dataset with a where condition                                                                             
    3) since hash is an "execution time object" it can be deleted and recreated for each observation.                                        
                                                                                                                                             
    options nonotes;                                                                                                                         
    data testc3;                                                                                                                             
      set                                                                                                                                    
        testc                                                                                                                                
      curobs=curobs indsname=indsname end=end                                                                                                
      ;                                                                                                                                      
                                                                                                                                             
      if missing(code) then                                                                                                                  
        do;                                                                                                                                  
          output;                                                                                                                            
        end;                                                                                                                                 
      else                                                                                                                                   
        do;                                                                                                                                  
          length strX $ 1024;                                                                                                                
          strX = cat(                                                                                                                        
            strip(indsname)                                                                                                                  
            , '(obs = ', curobs                                                                                                              
            , ' firstobs = ', curobs                                                                                                         
            , ' where = (', strip(code), '))'                                                                                                
            );                                                                                                                               
          declare hash H(dataset:strX);                                                                                                      
          rc = H.defineKey("code");                                                                                                          
          rc = H.defineDone();                                                                                                               
          if H.num_items > 0 then output;                                                                                                    
          rc = H.delete();                                                                                                                   
        end;                                                                                                                                 
                                                                                                                                             
      if end then put _N_=;                                                                                                                  
    run;                                                                                                                                     
    options notes;                                                                                                                           
                                                                                                                                             
    that one also isn't to fast but it took 3.41 seconds for 1000 obs.                                                                       
    and 38.84 seconds for 10000. so - much better.                                                                                           
                                                                                                                                             
    Bottom line is that: hash table "execution time nature" gives us                                                                         
    possibility to evaluate expressions embedded inside data at the run time - cool :-)                                                      
                                                                                                                                             
    All the best                                                                                                                             
    Bart                                                                                                                                     
                                                                                                                                             
                                                                                                                                             

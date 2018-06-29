# utl_determining_which_variable_produced_the_maximum_value
Determining which variable produced the maximum value.  Keywords: sas sql join merge big data analytics macros oracle teradata mysql sas communities stackoverflow statistics artificial inteligence AI Python R Java Javascript WPS Matlab SPSS Scala Perl C C# Excel MS Access JSON graphics maps NLP natural language processing machine learning igraph DOSUBL DOW loop stackoverflow SAS community.

    Determining which variable produced the maximum value

    see
    https://tinyurl.com/y8xcdfob
    https://github.com/rogerjdeangelis/utl_determining_which_variable_produced_the_maximum_value

    Same result WPS and SAS

    For other interesting solutions see.

      Only two solutions  handle multiple max values

         1. proc transpose (not shown)
         2. The one below

      Added Solutions (see end of message)

         3. Paul Dorfman hash
         4. Bart Jablonski <yabwon@gmail.com>  improved datastep
         
         
      FINAL: Nice sumary Paul
      Paul Dorfman via listserv.uga.edu



    https://tinyurl.com/ycyl4oxl
    https://communities.sas.com/t5/Base-SAS-Programming/Determining-which-variable-produced-the-maximum-val/m-p/473348

    SuryaKiran profile
    https://communities.sas.com/t5/user/viewprofilepage/user-id/83078


    INPUT
    =====

     WORK.HAVE total obs=100

      VAL1  VAL2  VAL3  VAL4  VAL5  VAL6  VAL7  VAL8  VAL9  VAL10 |  RULES
                                                                  |
      12     4    19     4    12     4     1     5    22      7   |  VAL9           -- 9 is the max
       1    22     4    45    48    36    20    18    16     25   |  VAL5
      14    17    12    21     6    35     0    25    45     11   |  VAL9
      39     7     5    26    32    38    27    48    26     21   |  VAL8
      24    47    35    17     9    43     6    15    48     28   |  VAL9
      15    32    27    34    30    12    17    13    36     45   |  VAL10
      30    35     1    35    16    17    26    35    21      8   |  VAL2|VAL4|VAL8 -- 2,4 and 8 = 35
      41    36     7    26    19    45    13    29    15     15   |  VAL6
      14    18    28    49     5    44     3     2    39     46   |  VAL4
      42    20     0    47    48     3    48    43     8     41   |  VAL5|VAL7      -- 5 and 7 = 48


    PROCESS
    =======

    data want;
      length Var $50.;
      set have;
       array vals (*) val1-val10;
       max_val=max(of  val1-val10);
       do i=1 to dim(vals);
           if vals(i)=max_val then var=CATX('|',strip(var),vname(vals(i)));
       end;
    drop i;
    run;quit;


    OUTPUT
    ======
    see above

    *                _               _       _
     _ __ ___   __ _| | _____     __| | __ _| |_ __ _
    | '_ ` _ \ / _` | |/ / _ \   / _` |/ _` | __/ _` |
    | | | | | | (_| |   <  __/  | (_| | (_| | || (_| |
    |_| |_| |_|\__,_|_|\_\___|   \__,_|\__,_|\__\__,_|

    ;

    data have;
      array vals[10] val1-val10;
      do rec=1 to 100;
         do i=1 to 10;
           vals[i]=int(50*uniform(1234));
         end;
         output;
      end;
      drop rec i;
    run;quit;

    *          _       _   _
     ___  ___ | |_   _| |_(_) ___  _ __
    / __|/ _ \| | | | | __| |/ _ \| '_ \
    \__ \ (_) | | |_| | |_| | (_) | | | |
    |___/\___/|_|\__,_|\__|_|\___/|_| |_|

    ;

    %utl_submit_wps64('
    libname wrk sas7bdat "%sysfunc(pathname(work))";
    data want;
      length Var $50.;
      set wrk.have;
       array vals (*) val1-val10;
       max_val=max(of  val1-val10);
       do i=1 to dim(vals);
           if vals(i)=max_val then var=CATX("|",strip(var),vname(vals(i)));
       end;
    drop i;
    run;quit;
    proc print;
    run;quit;
    ');

    *                                     _   _               _
     _ __   _____      __  _ __ ___   ___| |_| |__   ___   __| |___
    | '_ \ / _ \ \ /\ / / | '_ ` _ \ / _ \ __| '_ \ / _ \ / _` / __|
    | | | |  __/\ V  V /  | | | | | |  __/ |_| | | | (_) | (_| \__ \
    |_| |_|\___| \_/\_/   |_| |_| |_|\___|\__|_| |_|\___/ \__,_|___/

    ;


    Paul

    Roger,

    Your approach is the simplest. But note that effectively, it requires 2 passes through the array.
    The WANT step below is obviously less concise, yet its idea is almost as simple: Stick the values
    into a sorted hash stack, and then pop the needed items off its top.

    Effectively, the array is read once, and the only items retrieved from the hash table are the ones
    with the maximum value. Trade-off is the need to move the iterator pointer out of the table using
    the pair of LAST/NEXT method calls before it can be purged using the CLEAR method.

    Also note that the table has to be keyed using (key:-_i_) to ensure the proper concatenation order.
    A curious side effect of this ruse is that the values of variables _I_ in the key and data
    portions are different in every hash item.

    data have ;
      array val val1-val10 ;
      do _n_ = 1 to 100 ;
         do over val ;
           val = int (50 * uniform (1234)) ;
         end ;
         output ;
      end ;
    run ;

    data want (drop=_:) ;
      if _n_ = 1 then do ;
        dcl hash h (ordered:"d") ;
        h.definekey ("_n_", "_i_") ;
        h.definedone () ;
        dcl hiter hi ("h") ;
      end ;
      set have ;
      array vv val: ;
      do over vv ;
        h.add (key:vv, key:-_i_, data:vv, data:_i_) ;
        _vmax = _vmax max vv ;
      end ;
      length var $ 50. ;
      do while (hi.next() = 0 and _n_ = _vmax) ;
        var = catx ("|", var, cats ("var", _i_)) ;
      end ;
      _n_ = hi.last() ;
      _n_ = hi.next() ;
      h.clear() ;
    run ;

    Best regards

    Bart
    =====

    Bart Jablonski <yabwon@gmail.com>
    8:17 AM (40 minutes ago)
     to SAS-L, me
    Hi Roger and Paul,

    I think it can be done with one array reading (code below), or am I missing something?

    all the best
    Bart


    /*
    data have;
    input VAL1 VAL2 VAL3 VAL4 VAL5 VAL6 VAL7 VAL8 VAL9 VAL10;
    cards;
    12 4 19 4 12 4 1 5 22 7
    1 22 4 45 48 36 20 18 16 25
    14 17 12 21 6 35 0 25 45 11
    39 7 5 26 32 38 27 48 26 21
    24 47 35 17 9 43 6 15 48 28
    15 32 27 34 30 12 17 13 36 45
    30 35 1 35 16 17 26 35 21 8
    41 36 7 26 19 45 13 29 15 15
    14 18 28 49 5 44 3 2 39 46
    42 20 0 47 48 3 48 43 8 41
    1 1 1 1 1 1 1 1 1 1 1 1
    ;
    run;

    data want;
    set have;
    array V VAL:;
    length rules $ %sysevalf(10*5 + 1);

    max=.;

    do over V;
     if v>max then
      do;
      max = v;
      rules=strip(vname(V));
     end;
     else if v=max then
            rules=catx("|",rules,vname(V));
    end;


    run;
    
        %let D =  10 ;
    %let N = 100 ;

    data have ;
      array val val1 - val&d ;
      do _n_ = 1 to &n ;
         do over val ;
           val = int (50 * uniform (1234)) ;
         end ;
         output ;
      end ;
    run ;

    data want (drop = _:) ;
      set have ;
      array v val: ;
      length var $ %sysevalf (&d * 5 + 1) ;
      array vv [&d] $ 5 _temporary_ ;
      do over v ;
        if v > _m then do ;
          _m = v ;
          _j = 1 ;
          vv[_j] = vname (v) ;
        end ;
        else if v = _m then do ;
          _j + 1 ;
          vv[_j] = vname (v) ;
        end ;
      end ;
      do _i_ = 1 to _j ;
        var = catx ("|", var, vv[_i_]) ;
      end ;
    run ;



    Keintz, Mark via listserv.uga.edu
    11:20 AM (9 minutes ago)
     to SAS-L
    Paul, Bartosz

    You can achieve a little better performance by eschewing unneeded use of
    the vname function inside the do over loop.  Instead retrieve names from a
    temporary array created at _N_=1.

    Paul has regularly mentioned the superiority of key-index to other lookup structures.
    In this case it's also 5 times faster than vname function, per the code below:


    %let nv=80;

    data _null_;
      array V val1-val&nv;
      array vn [&nv] _temporary_;
      do over V;
        vn[_i_]=vname(V),
      end;
      do i=1 to 2**14;
         do over V;
           vnam=vn[_i_];
         end;
      end:
    run;

    ======
    SUMMARY
    =======

    Nice sumary Paul
    Paul Dorfman via listserv.uga.edu


    Bart,

    You sure do! More elegance + better performance in one package.
    I've been wondering about the hanging choice between >, <, and =, and you've nailed it.

    Please do permit me to add a bit more elegance by making your code
    still terser and a tiny bit of performance by eliminating the max assignment
    (it's not needed since max is set to missing at the top of the implied loop), like so:

    data wantB (keep = var) ;
      set have ;
      array v val: ;
      length var $ %sysevalf (&d * 5 + 1) ;
      do over v ;
        if v < _m then continue ;
        if v > _m then var = "" ;
        _m = v ;
        var = catx("|", var, vname(v)) ;
      end ;
    run ;

    Now your improvement has clued me in to do the same with my "concat at the end" code:

    data wantD (keep = var) ;
      set have ;
      array v val: ;
      length var $ %sysevalf (&d * 5 + 1) ;
      array vv [&d] $ 5 _temporary_ ;
      do over v ;
        if v < _m then continue ;
        if v > _m then _j = 0 ;
        _j + 1 ;
        vv[_j] = vname (v) ;
        _m = v ;
      end ;
      do _i_ = 1 to _j ;
        var = catx ("|", var, vv[_i_]) ;
      end ;
    run ;

    Unsurprisingly, this approach performs, relative to the rest, the better,
    the greater the dimension D is.

    However, with large D's the improvement suggested by Mark Keintz makes a far greater
    impact because it eschews the need of repeatedly calling the rather slow
    VNAME function in favor of pre-storing the variable names in a key-indexed table.
    For example, your code above with this augmentation would look like:

    data wantBM (keep = var) ;
      set have ;
      array v val: ;
      array vn [&d] $ 5 _temporary_ ;
      if _n_ = 1 then do over v ;
        vn[_i_] = vname (v) ;
      end ;
      length var $ %sysevalf (&d * 5 + 1) ;
      do over v ;
        if v < _m then continue ;
        if v > _m then var = "" ;
        _m = v ;
        var = catx("|", var, vn[_i_]) ;
      end ;
    run ;

    Needless to say, Roger's original code can use the same improvement:

    data wantRM (keep = var);
      set have ;
      array v val: ;
      length var $ %sysevalf (&d * 5 + 1) ;
      array vn [&d] $ 5 _temporary_ ;
      if _n_ = 1 then do over v ;
        vn[_i_] = vname (v) ;
      end ;
      _m = max (of v[*]) ;
       do over v ;
         if v = _m then var = catx ('|', var, vn[_i_]) ;
       end ;
    run ;

    To see how much performance different improvements add, I ran the code
    below for D=6000 and N=10000. For the purity of experiment, I had _NULL_ed
    out the output, then subtracted the pure read time from the resulting run
    times. This resulted in the following (in seconds):

    Roger original: 10.31
    Roger + Mark: 6.65
    Bart improved: 10.8
    Bart improved + Mark: 6.99
    Paul with Bart improvement: 10.3
    Paul with Bart improvement + Mark: 6.62

    Key-indexing the var names per Mark suggestion shaves about 30% off the run
    time no matter what, so obviously VNAME here is the biggest fish to fry.

    It's been fun ;).

    Best regards,
    Paul Dorfman

    %let d =  6000 ;
    %let n = 10000 ;

    data have ;
      array val val1 - val&d ;
      do _n_ = 1 to &n ;
         do over val ;
           val = int (50 * uniform (1234)) ;
         end ;
         output ;
      end ;
    run ;

    ** Pure read to detect time ** ;

    data _null_ ;
      set have ;
    run ;

    ** Roger original ** ;

    data _null_ ;
      set have ;
      array v val: ;
      length var $ %sysevalf (&d * 5 + 1) ;
      _m = max (of v[*]) ;
       do over v ;
         if v = _m then var = catx ('|', var, vname (v)) ;
       end ;
    run ;

    ** Roger + Mark ** ;

    data _null_ ;
      set have ;
      array v val: ;
      length var $ %sysevalf (&d * 5 + 1) ;
      array vn [&d] $ 5 _temporary_ ;
      if _n_ = 1 then do over v ;
        vn[_i_] = vname (v) ;
      end ;
      _m = max (of v[*]) ;
       do over v ;
         if v = _m then var = catx ('|', var, vn[_i_]) ;
       end ;
    run ;

    ** Bart improved ** ;

    data _null_ ;
      set have ;
      array v val: ;
      length var $ %sysevalf (&d * 5 + 1) ;
      do over v ;
        if v < _m then continue ;
        if v > _m then var = "" ;
        _m = v ;
        var = catx("|", var, vname(v)) ;
      end ;
    run ;

    ** Bart improved + Mark ** ;

    data _null_ ;
      set have ;
      array v val: ;
      array vn [&d] $ 5 _temporary_ ;
      if _n_ = 1 then do over v ;
        vn[_i_] = vname (v) ;
      end ;
      length var $ %sysevalf (&d * 5 + 1) ;
      do over v ;
        if v < _m then continue ;
        if v > _m then var = "" ;
        _m = v ;
        var = catx("|", var, vn[_i_]) ;
      end ;
    run ;

    ** Paul with Bart improvement ** ;

    data _null_ ;
      set have ;
      array v val: ;
      length var $ %sysevalf (&d * 5 + 1) ;
      array vv [&d] $ 5 _temporary_ ;
      do over v ;
        if v < _m then continue ;
        if v > _m then _j = 0 ;
        _j + 1 ;
        vv[_j] = vname (v) ;
        _m = v ;
      end ;
      do _i_ = 1 to _j ;
        var = catx ("|", var, vv[_i_]) ;
      end ;
    run ;

    ** Paul with Bart improvement + Mark ** ;

    data _null_ ;
      set have ;
      array v val: ;
      length var $ %sysevalf (&d * 5 + 1) ;
      array vn [&d] $ 5 _temporary_ ;
      if _n_ = 1 then do over v ;
        vn[_i_] = vname (v) ;
      end ;
      array vv [&d] $ 5 _temporary_ ;
      do over v ;
        if v < _m then continue ;
        if v > _m then _j = 0 ;
        _j + 1 ;
        vv[_j] = vn[_i_] ;
        _m = v ;
      end ;
      do _i_ = 1 to _j ;
        var = catx ("|", var, vv[_i_]) ;
      end ;
    run ;







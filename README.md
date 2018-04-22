# utl_speeding_up_a_moderate_size_interveave_of_tables
Speeding up a moderate size interveave of tables. Keywords: sas sql join merge big data analytics macros oracle teradata mysql sas communities stackoverflow statistics artificial inteligence AI Python R Java Javascript WPS Matlab SPSS Scala Perl C C# Excel MS Access JSON graphics maps NLP natural language processing machine learning igraph DOSUBL DOW loop stackoverflow SAS community.
    Speeding up a moderate size interveave of tables.

    github
    https://tinyurl.com/yan6rkve
    https://github.com/rogerjdeangelis/utl_speeding_up_a_moderate_size_interveave_of_tables

    The op has four datasets he needs to interleave by a single numeric key;
    Four of the datasets are small < 8 million rows and one is moderate size 280 million rows(480 columns).
    Possibly 480 variables in each file.

    QUESTION:  Is there a faster way to do the interleave?

    see
    https://tinyurl.com/yd29brdo
    https://communities.sas.com/t5/Base-SAS-Programming/Combining-multiple-SAS-datasets-while-maintaining-sort-order/m-p/456052

    SOME SUGGESTIONS

    You might experiment with SGIO and a large number of buffers - depends on locclity of reference?
    Dou you have striped raid 0 and SSDs?
    PCIE_X raid controller?
    On a mainframe keyed sequential VSAM can chop big flat files(like csvs) quickly?
    Unless you have a robust I/O system SPDS/SPDE may not help?


    POSSIBLE SOLUTION

    If the numeric key is not badly skewed, then do the following when segmenting the 280 million.
    You do need some information about the skew of the numeric key, maybe use a previous run;
    If skewed and you need to know the distribution amybe from a previous run you can cut appropreately.
    This is what teradata does.

    I suspect the problem centers around the 280 million csv.
    Split the 280 into 100 datasets(or more) instead of 25.
    It is not unusual for SPDE to partition into 100 peices (striped raid)

    Somehow you split the big file into 25 smaller files and mutitasked it.
    I have some thought s on this too.

    Suppose the following  gives you a somewhat uniform split of the numeric key

      100000000   (minimum key)
      200000000
      300000000
      >400000000

    * Make the 100 splits of the 280 million.

    * add the code below to each of the 25 spliting tasks - I suspect ypu mutitasked the split somehow;
    * a1 represents one of your 25 mutitasks.;

    data a1_1 a2_1 a3_1 a3_1;

        your existing code for the 280 million that creates one of the 25;

        select;
          when  ( key < 1000000001 )  output a1_1;
          when  ( key < 2000000001 )  output a2_1;
          when  ( key < 3000000001 )  output a3_1;
          otherwise output a4_3;
        end;

    run;quit;

    ...

    data a1_25 a2_25 a3_25 a4_25;

        your existing code for the 280 million that creates one of the 25;

        select;
          when  ( key < 1000000001 )  output a1_25;
          when  ( key < 2000000001 )  output a2_25;
          when  ( key < 3000000001 )  output a3_25;
          otherwise output a25_4;
        end;

    run;quit;

    Run four interleaves or more)

    * task 1;
    data grp1;

       set a1_:
           a1_:
           b(where=( key < 1000000001)
           c(where=( key < 1000000001)
           d(where=( key < 1000000001)
       ;
       by key;

    run;quit;
    ....

    data grp2;

       set
           a1_:
           b(where=( key < 2000000001)
           c(where=( key < 2000000001)
           d(where=( key < 2000000001)
       ;
       by key;

    run;quit;

    * do not create the dataset;
    * view should be almos as fast as a dataset?
    * having the pieces on separated disks may be a benfefit when doing analyticcs:
    
* do not know about recency and locality reference relation to key;


    data combine/(view=combine);
      set
         group:
    run;quit;


# utl-scraping-pdf-output-for-pdf-tables-and-lists
Scraping pdf output for pdf tables and lists
    Scraping pdf output for pdf tables and lists

       Two Related SAS/IML/R WPS/PROC RSolutions

            1. pdftools packahes (a list over 15 tables)
            2. tm package (a pdf table)


    github
    https://tinyurl.com/y8v7pvp2
    https://github.com/rogerjdeangelis/utl-scraping-pdf-output-for-pdf-tables-and-lists

    https://tinyurl.com/yal3n76r
    https://github.com/rogerjdeangelis/utl_convert_pdf_tables_to_SAS_WPS_datasets

    StckOverflow (see for full documentation)
    https://stackoverflow.com/questions/53898720/cleaning-data-from-pdf-file

    Julius Vainora profile
    https://stackoverflow.com/users/1320535/julius-vainora


    Related

    SAS FORUM (2 posts)
    https://tinyurl.com/y93699d2
    https://communities.sas.com/t5/SAS-Text-and-Content-Analytics/Need-to-extract-data-from-pdf-file/m-p/483338

    https://tinyurl.com/ycl2otqy
    https://communities.sas.com/t5/Base-SAS-Programming/PDF-to-SAS-Dataset/m-p/401636

    you may need to install Xpdf - I did not need it.
    http://www.xpdfreader.com/download.html


    INPUT
    =====

     https://www.ftse.com/products/downloads/FTSE_100_Constituent_history.pdf

     Page 1 of 15
                                                      ******
                                                    ***    ***
      FTSE 100                                     ** FTSE  **
      Historic Additions and Deletions             *          *
                                                   *  RUSSEL  *
                                                   ***      ***
                                                     ********

      Page 2 of 15


                                                      ******
                                                    ***    ***
      FTSE 100 Historic Additions and Deletions    ** FTSE  **
      ------------------------------------------   *          *
                                                   *  RUSSEL  *
                                                   ***      ***
                                                     ********


       Date       Added                        Deleted                     Notes
     -----------------------------------------------------------------------------------------------------------------

     19-Jan-84    Charterhouse J Rothschild    Eagle Star                  Corporate Event - Acquisition of Eagle Star
     02-Apr-84    Lonrho                       Magnet & Southerns
     02-Jul-84    Reuters                      Edinburgh Investment Trust
     02-Jul-84    Woolworths                   Barratt Development
     19-Jul-84    Enterprise Oil               Bowater Corporation
     01-Oct-84    Willis Faber                 Wimpey (George) & Co
    ....


    EXAMPLE OUTPUT
    --------------

    WORK.WANT total obs=460

       DATE       ADDED                                DELETED                     NOTES

     22-Apr-86    RMC Group                            Distillers Company          Corporate Event - Acquisition of Eagle Star
     01-Jul-86    British Printing & Communications    Abbey Life
     01-Jul-86    Burmah Oil                           Bank of Scotland  PROCESS
    ....


    PROCESS (see link for explanation)
    ==================================

    %utl_submit_r64('
     require(pdftools);
     require(data.table);
     require(stringr);
     require(SASxport);
     url <- "https://www.ftse.com/products/downloads/FTSE_100_Constituent_history.pdf";
     dfl <- pdf_text(url);
     dfl <- dfl[2:(length(dfl) - 1)];
     dfl <- gsub("\nFTSE Russell \\| FTSE 100 – Historic Additions and Deletions, November 2018[ ]+?\\d{1,2} of 12\n", "", dfl);
     dfl <- str_split(dfl, pattern = "(\n)(?=\\d{2}-\\w{3}-\\d{2})");
     dfl <- lapply(dfl, function(df) {
       df <- str_split_fixed(df, "(\n)*[ ]{2,}", 4);
       df <- gsub("(\n)*[ ]{2,}", " ", df);
       colnames(df) <- c("Date", "Added", "Deleted", "Notes");
       df[df == ""] <- NA;
       data.frame(df[-1, ]);
     });
     dfl<-do.call(rbind,dfl);
     str(dfl);
     dfl[]<-lapply(dfl, function(x) if(is.factor(x)) as.character(x) else x);
     str(dfl);
     write.xport(dfl,file="d:/xpt/want.xpt");
    ');

    libname xpt xport "d:/xpt/want.xpt";
    data want;
      set xpt.dfl;
    run;quit;
    libname xpt clear;

    LOG
    ---

    'data.frame':	460 obs. of  4 variables:
     $ Date   : chr  "19-Jan-84" "02-Apr-84" "02-Jul-84" "02-Jul-84" ...
     $ Added  : chr  "Charterhouse J Rothschild" "Lonrho" "Reuters" "Woolworths" ...
     $ Deleted: chr  "Eagle Star" "Magnet & Southerns\r" "Edinburgh Investment Trust\r" "Barratt Development\r" ...
     $ Notes  : chr  "Corporate Event - Acquisition of Eagle Star by\r BAT Industries\r" NA NA NA ...
    >
    NOTE: 16 lines were written to file PRINT.
    Stderr output:
    Loading required package: pdftools
    Warning message:
    package 'pdftools' was built under R version 3.5.2
    Loading required package: data.table
    Warning message:
    package 'data.table' was built under R version 3.5.2
    Loading required package: stringr
    Warning message:
    package 'stringr' was built under R version 3.5.2
    Loading required package: SASxport
    NOTE: 12 records were read from the infile RUT.
          The minimum record length was 2.
          The maximum record length was 793.
    NOTE: DATA statement used (Total process time):
          real time           11.76 seconds
          user cpu time       0.01 seconds
          system cpu time     0.12 seconds
          memory              268.40k
          OS Memory           19432.00k
          Timestamp           12/27/2018 01:26:02 PM
          Step Count                        102  Switch Count  0


    MPRINT(UTL_SUBMIT_R64):   filename rut clear;
    NOTE: Fileref RUT has been deassigned.
    MPRINT(UTL_SUBMIT_R64):   filename r_pgm clear;
    NOTE: Fileref R_PGM has been deassigned.
    MPRINT(UTL_SUBMIT_R64):   * use the clipboard to create macro variable;
    SYMBOLGEN:  Macro variable RETURNVAR resolves to N
    MLOGIC(UTL_SUBMIT_R64):  %IF condition %upcase(%substr(&returnVar.,1,1)) ne N is FALSE
    MLOGIC(UTL_SUBMIT_R64):  Ending execution.
    624   libname xpt xport "d:/xpt/want.xpt";
    NOTE: Libref XPT was successfully assigned as follows:
          Engine:        XPORT
          Physical Name: d:\xpt\want.xpt
    625   data want;
    626     set xpt.dfl;
    627   run;

    NOTE: There were 460 observations read from the data set XPT.DFL.
    NOTE: The data set WORK.WANT has 460 observations and 4 variables.
    NOTE: DATA statement used (Total process time):
          real time           0.01 seconds
          user cpu time       0.00 seconds
          system cpu time     0.01 seconds
          memory              483.87k
          OS Memory           19432.00k
          Timestamp           12/27/2018 01:26:02 PM
          Step Count                        103  Switch Count  0


    627 !     quit;
    628   libname xpt clear;
    NOTE: Libref XPT has been deassigned.


    OUTPUT
    ======
     see above



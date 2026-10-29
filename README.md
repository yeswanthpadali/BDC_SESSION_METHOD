# BDC_SESSION_METHOD

report ZBDCMM01PGM
       no standard page heading line-size 255.



TYPES: BEGIN OF TY_FILE,
        MATNR TYPE RMMG1-MATNR,
        MBRSH TYPE RMMG1-MBRSH,
        MTART TYPE RMMG1-MTART,
        WERKS TYPE RMMG1-WERKS,
        MAKTX TYPE MAKT-MAKTX,
        MEINS TYPE MARA-MEINS,
        MATKL TYPE MARA-MATKL,
        MEINS1 TYPE MARA-MEINS,
        EKGRP TYPE MARC-EKGRP,
        MATKL1 TYPE MARA-MATKL,
      END OF TY_FILE.

DATA: GT_FILE TYPE TABLE OF TY_FILE,
      GS_FILE TYPE TY_FILE,
      GT_RAW TYPE TRUXS_T_TEXT_DATA.
DATA: GS_BDCDATA TYPE BDCDATA,
      GT_BDCDATA TYPE TABLE OF BDCDATA.

parameters:p_file type rlgrap-filename.

AT SELECTION-SCREEN ON VALUE-REQUEST FOR P_FILE.

  CALL FUNCTION 'F4_FILENAME'
   EXPORTING
     PROGRAM_NAME        = SY-CPROG
     DYNPRO_NUMBER       = SY-DYNNR
*     FIELD_NAME          = ''
   IMPORTING
     FILE_NAME           = P_FILE.

  START-OF-SELECTION.
  CALL FUNCTION 'TEXT_CONVERT_XLS_TO_SAP'
    EXPORTING
*     I_FIELD_SEPERATOR          =
      I_LINE_HEADER              = 'X'
      I_TAB_RAW_DATA             = GT_RAW
      I_FILENAME                 = P_FILE
    TABLES
      I_TAB_CONVERTED_DATA       = GT_FILE
*   EXCEPTIONS
*     CONVERSION_FAILED          = 1
*     OTHERS                     = 2
            .
  IF SY-SUBRC <> 0.
MESSAGE 'SUCCESSFULLY FETCHED TO ITAB' TYPE 'S'.
  ENDIF.

CALL FUNCTION 'BDC_OPEN_GROUP'
 EXPORTING
*   CLIENT                    = SY-MANDT
*   DEST                      = FILLER8
   GROUP                     = 'z8mm01'
   HOLDDATE                  = sy-datum
   KEEP                      = 'X'
   USER                      = SY-UNAME
*   RECORD                    = FILLER1
*   PROG                      = SY-CPROG
*   DCPFM                     = '%'
*   DATFM                     = '%'
* IMPORTING
*   QID                       =
* EXCEPTIONS
*   CLIENT_INVALID            = 1
*   DESTINATION_INVALID       = 2
*   GROUP_INVALID             = 3
*   GROUP_IS_LOCKED           = 4
*   HOLDDATE_INVALID          = 5
*   INTERNAL_ERROR            = 6
*   QUEUE_ERROR               = 7
*   RUNNING                   = 8
*   SYSTEM_LOCK_ERROR         = 9
*   USER_INVALID              = 10
*   OTHERS                    = 11
          .
IF SY-SUBRC <> 0.
* Implement suitable error handling here
ENDIF.


LOOP AT GT_FILE INTO GS_FILE.


perform bdc_dynpro      using 'SAPLMGMM' '0060'.
perform bdc_field       using 'BDC_CURSOR'
                              'RMMG1-MATNR'.
perform bdc_field       using 'BDC_OKCODE'
                              '=ENTR'.
perform bdc_field       using 'RMMG1-MATNR'
                              GS_FILE-MATNR."'60000000'.
perform bdc_field       using 'RMMG1-MBRSH'
                              GS_FILE-MBRSH."'F'.
perform bdc_field       using 'RMMG1-MTART'
                              GS_FILE-MTART."'ROH'.


perform bdc_dynpro      using 'SAPLMGMM' '0070'.
perform bdc_field       using 'BDC_CURSOR'
                              'MSICHTAUSW-DYTXT(09)'.
perform bdc_field       using 'BDC_OKCODE'
                              '=ENTR'.
perform bdc_field       using 'MSICHTAUSW-KZSEL(01)'
                              'X'.
perform bdc_field       using 'MSICHTAUSW-KZSEL(02)'
                              'X'.
perform bdc_field       using 'MSICHTAUSW-KZSEL(09)'
                              'X'.
perform bdc_dynpro      using 'SAPLMGMM' '0080'.
perform bdc_field       using 'BDC_CURSOR'
                              'RMMG1-WERKS'.
perform bdc_field       using 'BDC_OKCODE'
                              '=ENTR'.
perform bdc_field       using 'RMMG1-WERKS'
                              '0099'.
perform bdc_dynpro      using 'SAPLMGMM' '4004'.
perform bdc_field       using 'BDC_OKCODE'
                              'ENTR'.
perform bdc_field       using 'MAKT-MAKTX'
                              GS_FILE-MAKTX."'Test BDC Material 1'.
perform bdc_field       using 'BDC_CURSOR'
                              'MARA-MATKL'.
perform bdc_field       using 'MARA-MEINS'
                              GS_FILE-MEINS."'EA'.
perform bdc_field       using 'MARA-MATKL'
                              GS_FILE-MATKL ."'0001'.
perform bdc_dynpro      using 'SAPLMGMM' '4004'.
perform bdc_field       using 'BDC_OKCODE'
                              '/00'.
perform bdc_field       using 'BDC_CURSOR'
                              'MAKT-MAKTX'.
perform bdc_field       using 'MAKT-MAKTX'
                              GS_FILE-MAKTX."'Test BDC Material 1'.
perform bdc_dynpro      using 'SAPLMGMM' '4000'.
perform bdc_field       using 'BDC_OKCODE'
                              '/00'.
perform bdc_field       using 'MAKT-MAKTX'
                              GS_FILE-MAKTX."'Test BDC Material 1'.
perform bdc_field       using 'BDC_CURSOR'
                              'MARA-MATKL'.
perform bdc_field       using 'MARA-MEINS'
                              GS_FILE-MEINS1."'EA'.
perform bdc_field       using 'MARC-EKGRP'
                              GS_FILE-EKGRP."'000'.
perform bdc_field       using 'MARA-MATKL'
                              GS_FILE-MATKL1."'0001'.
perform bdc_dynpro      using 'SAPLSPO1' '0300'.
perform bdc_field       using 'BDC_OKCODE'
                              '=YES'.


CALL FUNCTION 'BDC_INSERT'
 EXPORTING
    TCODE                  = 'MM01'
*   POST_LOCAL             = NOVBLOCAL
*   PRINTING               = NOPRINT
*   SIMUBATCH              = ' '
*   CTUPARAMS              = ' '
  TABLES
    DYNPROTAB              = GT_BDCDATA
* EXCEPTIONS
*   INTERNAL_ERROR         = 1
*   NOT_OPEN               = 2
*   QUEUE_ERROR            = 3
*   TCODE_INVALID          = 4
*   PRINTING_INVALID       = 5
*   POSTING_INVALID        = 6
*   OTHERS                 = 7
          .
IF SY-SUBRC <> 0.
* Implement suitable error handling here
ENDIF.

REFRESH:GT_BDCDATA.

ENDLOOP.

CALL FUNCTION 'BDC_CLOSE_GROUP'
* EXCEPTIONS
*   NOT_OPEN          = 1
*   QUEUE_ERROR       = 2
*   OTHERS            = 3
          .
IF SY-SUBRC <> 0.
* Implement suitable error handling here
ENDIF.


*&---------------------------------------------------------------------*
*&      Form  BDC_DYNPRO
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*      -->P_0130   text
*      -->P_0131   text
*----------------------------------------------------------------------*
FORM BDC_DYNPRO  USING    VALUE(P_0130)
                          VALUE(P_0131).

  GS_BDCDATA-PROGRAM = P_0130.
  GS_BDCDATA-DYNPRO  = P_0131.
  GS_BDCDATA-DYNBEGIN = 'X'.
  APPEND GS_BDCDATA TO GT_BDCDATA.
  CLEAR GS_BDCDATA.


ENDFORM.
*&---------------------------------------------------------------------*
*&      Form  BDC_FIELD
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*      -->P_0145   text
*      -->P_0146   text
*----------------------------------------------------------------------*
FORM BDC_FIELD  USING    VALUE(P_0145)
                         VALUE(P_0146).

  GS_BDCDATA-FNAM = P_0145.
  GS_BDCDATA-FVAL  = P_0146.

  APPEND GS_BDCDATA TO GT_BDCDATA.
  CLEAR GS_BDCDATA.

ENDFORM.

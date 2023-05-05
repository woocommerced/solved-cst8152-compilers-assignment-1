Download Link: https://assignmentchef.com/product/solved-cst8152-compilers-assignment-1
<br>
<h1><u>Purpose:</u> Programming and Using Dynamic Structures (buffers) with C</h1>




This is a review of and an exercise in C coding style, programming techniques, data types and structures, memory management, and simple file input/output.  It will give you a better understanding of the type of internal data structures used by a simple compiler you will be building this semester. This assignment will be also an exercise in “<em>excessively</em> <em>defensive programming</em>”. You are to write functions that should be “overly” protected and should not abruptly terminate or “<strong>crash</strong>” at run-time due to invalid function parameters, erroneous internal calculations, or memory violations. To complete the assignment, you should fulfill the following two tasks:

<strong> </strong>

<h1><u>Task 1:</u> The Buffer Data Structure and Utility Functions</h1>




Buffers are often used when developing compilers because of their efficiency (see page 111 of your textbook). You are to implement a buffer that can operate in three different modes: a “<strong>f</strong>ixedsize” buffer, an “<strong>a</strong>dditive self-incrementing” buffer, and a “<strong>m</strong>ultiplicative self-incrementing” buffer. The buffer implementation is based on two associated data structures: a <strong><em>Buffer Descriptor </em></strong>(or <em>Buffer Handle)</em> and an array of characters (the actual character buffer). Both structures are to be created “on demand” at run time, that is, they are to be allocated dynamically. The <strong><em>Buffer </em></strong>

<strong><em>Descriptor</em></strong> or <strong><em>Buffer Handle</em></strong> – the names suggest the purpose of this buffer control data structure – contains all the necessary information about the array of characters: a pointer to the beginning of the character array location in memory, the current capacity, the next character entry position, the increment factor, the operational mode and some additional parameters.




In this assignment you are to complete the coding for a “<em>buffer utility</em>“, which includes the buffer data structure and the associated functions, following strictly the given specifications. Use the data declarations and function prototypes given below.<strong> Do not change the names and the data types of the functions and the variables. Any change will be regarded as a serious specification violation.</strong> Write the associated code.

The following structure declaration must be used to implement the Buffer Descriptor:




typedef struct BufferDescriptor {

char *cb_head;        /* pointer to the beginning of character array (character buffer) */         short capacity;         /* current dynamic memory size (in bytes) allocated to character buffer */         short addc_offset;   /* the offset (in chars) to the add-character location */         short getc_offset;    /* the offset (in chars) to the get-character location */         short markc_offset;   /* the offset (in chars) to the mark location */         char  inc_factor;      /* character array increment factor*/         char  mode;             /* operational mode indicator*/

unsigned short  flags;    /* contains character array reallocation flag and end-of-buffer flag */ } Buffer, *pBuffer;

Where:

<strong><em> </em></strong>

<strong><em>capacity </em></strong>is the current total size (measured in bytes) of the memory allocated for the character array by <strong><em>malloc()/realloc() </em></strong>functions. In the text below it is referred also as current capacity.  It is whatever value you have used in the call to <strong><em>malloc()/realloc()</em></strong> that allocates the storage pointed to by <strong><em>cb_head</em></strong>.




<strong><em>inc_factor </em></strong>is a buffer increment factor. It is used in the calculations of a new buffer <strong>capacity</strong> when the buffer needs to grow. The buffer needs to grow when it is full but still another character needs to be added to the buffer. The buffer is full when <strong><em>addc_offset</em></strong> measured in bytes is equal to <strong><em>capacity</em></strong> and thus all the allocated memory has been used. The<strong><em> inc_factor</em></strong> is only used when the buffer operates in one of the “self-incrementing” modes. In “<strong>a</strong>dditive self-incrementing” mode it is a positive integer number in the range of 1 to 255 and represents directly the increment (measured in characters) that must be added to the current capacity every time the buffer needs to grow. In “<strong>m</strong>ultiplicative self-incrementing” mode it is a positive integer number in the range of 1 to 100 and represents a percentage used to calculate the new capacity increment that must be added to the current capacity every time the buffer needs to grow.




<strong><em>addc_offset</em></strong> is the distance (measured in chars) from the beginning of the character array (<strong><em>cb_head</em></strong>) to the location where the next character is to be added to the existing buffer content. <strong><em>addc_offset</em></strong> (measured in bytes) must never be larger than <strong><em>capacity</em></strong>, or else you are overrunning the buffer in memory and your program may crash at run-time or destroy data.




<strong><em>getc_offset</em></strong> is the distance (measured in chars) from the beginning of the character array (<strong><em>cb_head</em></strong>) to the location of the character which will be returned if the function <strong><em>b_getc()</em></strong> is called. The value <strong><em>getc_offset</em></strong> (measured in chars) must never be larger than <strong><em>addc_offset</em></strong>, or else you are overrunning the buffer in memory and your program may get wrong data or crash at run-time. If the value of <strong><em>getc_offset </em></strong>is equal to the value of <strong><em>addc_offset</em></strong>, the buffer has reached the end of its current content.




<strong><em>markc_offset</em></strong> is the distance (measured in chars) from the beginning of the character array (<strong><em>cb_head</em></strong>) to the location of a <em>mark</em>. A <em>mark</em> is a location in the buffer, which indicates the position of a specific character (for example, the beginning of a word or a phrase).




<strong><em>mode </em></strong>is an operational mode indicator. It can be set to three different integer numbers: 1, 0, and  –1. The number <strong>0</strong> indicates that the buffer operates in “<strong>f</strong>ixed-size” mode; <strong>1</strong> indicates “<strong>a</strong>dditive selfincrementing” mode; and <strong>–1</strong> indicates “<strong>m</strong>ultiplicative self-incrementing” mode. The mode is set when a new buffer is created and cannot be changed later.




<strong><em>flags</em> </strong>is a field containing different flags and indicators. In cases when storage space must be as small as possible, the common approach is to pack several data items into single variable; one common use is a set of single-bit or multiple-bit flags or indicators in applications like compiler buffers, file buffers, and database fields. The flags are usually manipulated through different bitwise operations using a set of “masks.” Alternative technique is to use <em>bit-fields. </em>Using bit-fields allows individual fields to be manipulated in the same way as structure members are manipulated. Since almost everything about bit-fields is implementation dependent, this approach should be avoided if the portability is a concern. In this implementation, you are to use bitwise operations and masks (see <em>bitmask.c</em> example).




Each flag or indicator uses one or more bits of the <strong><em>flags</em></strong> field. The flags usually indicate that something happened during a routine operation (end of file, end of buffer, integer arithmetic sign overflow, and so on). Multiple-bit indicators can indicate, for example, the mode of the buffer (three different combinations – therefore 2-bits are needed). In this implementation the <strong><em>flags</em></strong> field has the following structure:







<table width="528">

 <tbody>

  <tr>

   <td width="54"><strong>MSB</strong> 15</td>

   <td width="36">14</td>

   <td width="30">13</td>

   <td width="30">12</td>

   <td width="30">11</td>

   <td width="30">10</td>

   <td width="24">9</td>

   <td width="30">8</td>

   <td width="24">7</td>

   <td width="24">6</td>

   <td width="24">5</td>

   <td width="24">4</td>

   <td width="24">3</td>

   <td width="33">2</td>

   <td width="50">   1</td>

   <td width="61">0   <strong>LSB</strong></td>

  </tr>

  <tr>

   <td width="54">     1</td>

   <td width="36">1</td>

   <td width="30">1</td>

   <td width="30">1</td>

   <td width="30">1</td>

   <td width="30">1</td>

   <td width="24">1</td>

   <td width="30">1</td>

   <td width="24">1</td>

   <td width="24">1</td>

   <td width="24">1</td>

   <td width="24">1</td>

   <td width="24">1</td>

   <td width="33">1</td>

   <td width="50">   x</td>

   <td width="61">x</td>

  </tr>

  <tr>

   <td colspan="13" width="384">reserved for future use must be set to 1 and stay 1 all the time in this implementation</td>

   <td width="33"> </td>

   <td width="50"><strong>r_flag </strong></td>

   <td width="61"><strong>eob</strong> flag</td>

  </tr>

 </tbody>

</table>

Bit

Contents

Description




The LSB bit (bit 0) of the <strong><em>flags</em> </strong>field is a single-bit <em>end-of-buffer </em>(<strong><em>eob</em></strong>) flag. The <strong><em>eob</em></strong> bit is by default <strong>0</strong>, and when set to 1, it indicates that the end of the buffer content has been reached during the buffer read operation (<strong><em>b_getc()</em></strong> function). If <strong><em>eob</em></strong> is set to <strong>1</strong>, the function <strong><em>b_getc()</em></strong> should not be called before the <strong><em>getc_offset</em></strong> is reset by another operation.




Bit 1 of the <strong><em>flags</em></strong> field is a single-bit reallocation flag (<strong><em>r_flag</em></strong>). The <strong><em>r_flag</em></strong> bit is by default <strong>0</strong>, and when set to 1, it indicates that the location of the buffer character array in memory has been changed due to memory reallocation. This could happen when the buffer needs to expand or shrink. The flag can be used to avoid dangling pointers when pointers instead of offsets are used to access the information in the character buffer.




The rest of the bits are reserved for further use and must be set by default to <strong>1 </strong>and they must not be changed by the bitwise operation manipulating bit 0 and bit 1.




You are to implement the following set of buffer utility functions (operations). Later they will be used by all other parts of the compiler when a temporary storage space is needed.

The <strong>first implementation step</strong> in all functions must be the validation (if possible and appropriate) of the function arguments. If an argument value is invalid, the function must return an appropriate failure indicator.




<h2>Buffer * b_allocate (short init_capacity,char inc_factor,char o_mode)</h2>

This function creates a new buffer in memory (on the program heap). The function

<ul>

 <li>tries to allocate memory for one <strong><em>Buffer</em></strong> structure using <strong><em>calloc()</em></strong>;</li>

 <li>tries to allocates memory for one dynamic character buffer (character array) calling<strong><em> malloc()</em></strong> with the given initial capacity <strong><em>init_capacity</em></strong>. The range of the parameter <strong><em>init_capacity </em></strong>must be between 0 and the <strong><em>MAXIMUM ALLOWED POSITIVE VALUE</em></strong><em> – 1 </em> The <em>maximum allowed positive value</em> is determined by the data type of the parameter <strong><em>init_capacity</em></strong>. The pointer returned by <strong><em>malloc()</em></strong> is assigned to <strong><em>cb_head</em></strong>;</li>

 <li>sets the buffer operational mode indicator <strong><em>mode </em></strong>and the Buffer structure increment factor<strong><em> inc_factor</em></strong>. If the <em>o_mode </em>is the symbol <strong><em>f</em></strong> or <em>inc_factor</em> is <strong>0, </strong>the <strong><em>mode</em></strong> and the buffer <strong><em>inc_factor</em></strong> are set to number <strong>0</strong>. If the <em>o_mode </em>is the symbol <strong><em>f</em></strong> and <em>inc_factor</em> is not <strong>0, </strong>the <strong><em>mode</em></strong> and the buffer<strong> <em>inc_factor</em></strong> are set to 0. If the <em>o_mode </em>is <strong><em>a</em></strong> and <em>inc_factor </em>is in the range of <strong>1</strong> to <strong>255 </strong>inclusive , the <strong><em>mode</em></strong>  is set to number <strong>1 </strong>and the buffer<strong> <em>inc_factor </em></strong>is set to the value of <em>inc_factor</em>. If the <em>o_mode </em> is <strong><em>m</em></strong>  and <em>inc_factor </em>is in the range of <strong>1</strong> to <strong>100 </strong>inclusive, the <strong><em>mode</em></strong> is set to number <strong>-1</strong> and the <em>inc_factor</em> value is assigned to the buffer<strong> <em>inc_factor</em></strong>; – copies the given <strong><em>init_capacity</em></strong> value into the Buffer structure<strong><em> capacity</em></strong> variable; – sets the <strong><em>flags</em></strong> field to its default value which is FFFC hexadecimal.</li>

</ul>




Finally, on success, the function returns a pointer to the <strong><em>Buffer</em></strong> structure. It must return <strong>NULL </strong>pointer on any error which violates the constraints imposed upon the buffer parameters or prevents the creation of a working buffer. If run-time error occurs, the function must return immediately after the error is discovered.  Check for all possible errors which can occur at run time. Do not allow

<strong>“memory leaks”,</strong> <strong>“dangling”</strong> pointers, or “<strong>bad</strong>” parameters.




<h2>pBuffer b_addc (pBuffer const pBD, char symbol)</h2>

Using a bitwise operation the function resets the <strong><em>flags</em></strong> field <strong><em>r_flag </em></strong>bit to 0 and tries to add the character <strong><em>symbol</em></strong> to the character array of the given <strong><em>buffer </em></strong>pointed by <strong><em>pBD</em></strong>. If the buffer is operational and it is not full, the symbol can be stored in the character buffer. In this case, the function adds the character to the content of the character buffer, increments <strong><em>addc_offset </em></strong>by 1 and returns.




If the character buffer is already full, the function will try to resize the buffer by increasing the current capacity to a new capacity. How the capacity is increased depends on the current operational mode of the buffer.




If the operational mode is <strong>0</strong>, the function returns NULL.




If the operational mode is <strong>1</strong>, it tries to increase the current capacity of the buffer to a <em>new capacity</em> by adding <strong><em>inc_factor </em></strong>(converted to bytes) to <strong><em>capacity</em></strong>. If the result from the operation is positive and does not exceed the <strong><em>MAXIMUM ALLOWED POSITIVE VALUE</em></strong><em> –1</em><em> (</em>minus<em> 1)</em>, the function proceeds. If the result from the operation is positive but exceeds the <strong><em>MAXIMUM ALLOWED POSITIVE VALUE</em></strong><em> –1</em><em> (</em>minus<em> 1)</em>, it assigns the <strong><em>MAXIMUM ALLOWED POSITIVE VALUE</em></strong><em> –1</em> to the <em>new capacity </em>and proceeds. The <strong><em>MAXIMUM ALLOWED POSITIVE VALUE</em></strong> is determined by the data type of the variable, which contains the buffer capacity.

If the result from the operation is negative, it returns NULL.







If the operational mode is <strong>-1</strong> it tries to increase the current capacity of the buffer to a <em>new capacity</em> in the following manner:




– If the current capacity can not be incremented anymore because it has already reached the maximum capacity of the buffer, the function returns NULL.




The function tries to increase the current capacity using the following formulae:

<em> </em>

<em>available space = maximum buffer capacity – current capacity new increment =  available space *  inc_factor  / 100</em> <em>new capacity = current capacity  +  new increment </em>




The <em>maximum buffer capacity </em>is the <strong><em>MAXIMUM ALLOWED POSITIVE VALUE</em></strong><em> –1.</em> If the <em>new capacity</em> has been incremented successfully, no further adjustment of the <em>new capacity</em> is required. If as a result of the calculations, the <em>current capacity </em>cannot be incremented, but the <em>current capacity</em> is still smaller than the <strong><em>MAXIMUM ALLOWED POSITIVE VALUE</em></strong><em> –1</em>, then the <strong><em>MAXIMUM ALLOWED POSITIVE VALUE</em></strong><em> –1</em> is assigned to the <em>new capacity </em>and the function proceeds<strong><em>.</em></strong>




If the capacity increment in mode <strong>1</strong> or<strong> -1</strong> is successful, the function performs the following operations:




<ul>

 <li>the function tries to expand the character buffer calling <strong><em>realloc()</em></strong> with the <em>new capacity</em>. If the reallocation fails, the function returns NULL;</li>

 <li>if the location in memory of the character buffer has been changed by the reallocation, the function sets <strong><em>r_flag</em></strong> bit to <strong>1</strong> using a bitwise operation;</li>

 <li>adds (appends) the character <strong><em>symbol</em></strong> to the buffer content;</li>

 <li>changes the value of <strong><em>addc_offset </em></strong>by 1, and saves the newly calculated capacity value into<strong><em> capacity</em></strong> variable;</li>

 <li>the function returns a pointer to the <strong>Buffer</strong></li>

</ul>




The function must return NULL on any error. Some of the possible errors are indicated above but you must check for all possible errors that can occur at run-time. Do not allow “<strong>memory leaks</strong>”.

Avoid creating “<strong>dangling pointers</strong>” and using “<strong>bad</strong>” parameters. The function <strong><em>must not destroy</em></strong> the buffer or the contents of the buffer even when an error occurs – it must simply return NULL leaving the existing buffer content intact. <strong>A change in the project platform (16-bit, 32-bit or 64bit) must not lead to improper behavior</strong>.




<h2>int b_clear (Buffer * const pBD)</h2>

The function retains the memory space currently allocated to the buffer, but re-initializes all appropriate data members of the given <strong><em>Buffer</em></strong> structure (buffer descriptor), such that the buffer will appear empty and the next call to <strong><em>b_addc()</em></strong> will put the character at the beginning of the character buffer. The function does not need to clear the existing contents of the character buffer. If a runtime error is possible, the function should return <strong>–1 </strong>in order to notify the calling function about the failure.




<h2>void b_free (Buffer * const pBD)</h2>

The function de-allocates (frees) the memory occupied by the character buffer and the <strong><em>Buffer</em></strong> structure (buffer descriptor). The function should not cause abnormal behavior (crash).




<h2>int b_isfull (Buffer * const pBD)</h2>

The function returns <strong><em>1 </em></strong>if the character buffer is full; it returns <strong><em>0</em></strong> otherwise. If a run-time error is possible, the function should return <strong>–1</strong>.

<strong><em> </em></strong>

<h2>short b_limit (Buffer * const pBD)</h2>

The function returns the current limit of the character buffer. The current limit is the amount of space measured in chars that is currently being used by all added (stored) characters. If a run-time error is possible, the function should return <strong>–1</strong>.




<h2>short b_capacity(Buffer * const pBD)</h2>

The function returns the current capacity of the character buffer. If a run-time error is possible, the function should return <strong>–1</strong>.




<h2>short b_mark (pBuffer const pBD, short mark)</h2>

The function sets <strong><em>markc_offset </em></strong>to<strong><em> mark</em></strong>. The parameter <strong><em>mark</em></strong> must be within the current limit of the buffer (0 to <strong><em>addc_offset </em></strong>inclusive). The function returns the currently set <strong><em>markc_offset</em></strong>. If a run-time error is possible, the function should return <strong>-1.</strong>




<h2>int b_mode (Buffer * const pBD)</h2>

The function returns the value of <strong><em>mode </em></strong>to the calling function. If a run-time error is possible, the function should notify the calling function about the failure .

<h2>size_t  b_incfactor (Buffer * const pBD)</h2>

The function returns the non-negative value of <strong><em>inc_factor </em></strong>to the calling function. If a run-time error is possible, the function should return <strong>0x100</strong>.

<strong><em> </em></strong>

<h2>int b_load (FILE * const fi, Buffer * const pBD)</h2>

The function loads (reads) an open input file specified by<strong><em> fi </em></strong>into a buffer specified by <strong><em>pBD</em></strong>. The function must use the standard function <strong><em>fgetc(fi)</em></strong> to read one character at a time and the function <strong><em>b_addc()</em></strong> to add the character to the buffer. If the current character cannot be added to the buffer, the function returns the character to the file stream (file buffer) using <strong><em>ungetc() </em></strong>library function, then prints the returned character both as a character and as an integer (see test file ass1fi.out) and then returns <strong>-2</strong> (use the defined <em>LOAD_FAIL</em> constant). The operation is repeated until the standard macro <strong><em>feof(fi)</em></strong> detects end-of-file on the input file. The end-of-file character must not be added to the content of the buffer.

Only the standard macro <strong><em>feof(fi) </em></strong>must be used to detect end-of-file on the input file. Using other means to detect end-of-file on the input file will be considered a significant specification violation. If some other run-time errors are possible, the function should return <strong>–1</strong>. If the loading operation is successful, the function must return the number of characters added to the buffer.

<strong><em> </em></strong>

<h2>int b_isempty (Buffer * const pBD)</h2>

If the <strong><em>addc_offset</em></strong> is 0, the function returns 1; otherwise it returns 0. If a run-time error is possible, it should return  <strong>–1</strong>.

<strong><em> </em></strong>

<h2>char b_getc (Buffer * const pBD)</h2>

This function is used to read the buffer. The function performs the following steps:

<ul>

 <li>checks the argument for validity (possible run-time error). If it is not valid, it returns <strong>-2</strong>;</li>

 <li>if <strong><em>getc_offset</em></strong> and <strong><em>addc_offset</em></strong> are equal, using a bitwise operation it sets the <strong><em>flags</em> </strong>field <strong><em>eob</em> </strong>bit to <strong>1</strong> and returns number <strong>0</strong>; otherwise, using a bitwise operation it sets <strong><em>eob</em></strong> to <strong>0</strong>;</li>

 <li>returns the character located at <strong><em>getc_offset</em></strong>. Before returning it increments <strong><em>getc_offset</em></strong> by <strong>1</strong>.</li>

</ul>




<h2>int b_eob (Buffer * const pBD)</h2>

The function returns the <strong><em>eob </em></strong>bit value to the calling function. A bitwise operation must be used to return the value of the <strong><em>flags</em></strong> field <strong><em>eob</em></strong> bit. If a run-time error is possible, it should return  <strong>–1</strong>.




<h2>int b_print (Buffer * const pBD)</h2>

The function is intended to be used for used for diagnostic purposes only. Using the <strong><em>printf() </em></strong>library function the function prints character by character the contents of the character buffer to the standard output (stdout). Before printing the content the function checks if the buffer is empty, and if it is, it prints the following message Empty buffer! adding a new line at the end and returns. Next, the function prints the content calling <strong><em>b_getc() </em></strong>in a loop and using <strong><em>b_eob() </em></strong>to detect the end of the buffer content. Finally, it prints a new line character. It returns the number of characters printed. The function returns <strong>–1 </strong>on failure.




<h2>Buffer * b_compact(Buffer * const pBD, char symbol)</h2>

For all operational modes of the buffer the function shrinks (or in some cases may expand) the buffer to a <em>new capacity</em>. The <em>new capacity</em> is the current limit plus a space for one more character. In other words the <em>new capacity</em> is <strong><em>addc_offset + 1</em></strong> converted to bytes. The function uses <strong><em>realloc()</em></strong> to adjust the <em>new capacity</em>, and then updates all the necessary members of the buffer descriptor structure. Before returning a pointer to <strong><em>Buffer</em></strong>, the function adds the <strong><em>symbol</em></strong> to the end of the character buffer (<strong>do not</strong> use <strong><em>b_addc()</em></strong>, use <strong><em>addc_offset </em></strong>to add the symbol) and increments<strong><em> addc_offset.</em></strong> The function must return NULL if for some reason it cannot to perform the required operation. It must set the <strong><em>r_flag </em></strong>bit appropriately<strong><em>.</em></strong>




<h2>char b_rflag (Buffer * const pBD)</h2>

The function returns the <strong><em>r_flag </em></strong>bit value to the calling function. A bitwise operation must be used to return the value of the <strong><em>flags</em></strong> field <strong><em>r_flag</em></strong> bit. If a run-time error is possible, it should return  <strong>–1</strong>.







<h2>short b_retract (Buffer * const pBD)</h2>

The function decrements <strong><em>getc_offset </em></strong>by <strong>1</strong>. If a run-time error is possible, it should return  <strong>–1</strong>; otherwise it returns <strong><em>getc_offset</em></strong>.




<h2>short b_reset (Buffer * const pBD)</h2>

The function sets <strong><em>getc_offset </em></strong>to the value of the current <strong><em>markc_offset</em></strong> . If a run-time error is possible, it should return  <strong>–1</strong>; otherwise it returns <strong><em>getc_offset</em></strong>.

<strong> </strong>

<h2>short b_getcoffset (Buffer * const pBD)</h2>

The function returns <strong><em>getc_offset </em></strong>to the calling function. If a run-time error is possible, it should return  <strong>–1</strong>.




<h2>int b_rewind(Buffer * const pBD)</h2>

The function set the <strong><em>getc_offset </em></strong>and<strong><em> markc_offset</em></strong> to 0, so that the buffer can be reread again. If a run-time error is possible, it should return  <strong>–1</strong>; otherwise it returns 0;




<h2>char * b_location(Buffer * const pBD, short loc_offset)</h2>

The function returns a pointer to a location of the character buffer indicated by <strong><em>loc_offset</em></strong>.  <strong><em>loc_offset </em></strong>is the distance (measured in chars) from the beginning of the character array (<strong><em>cb_head</em></strong>). If a run-time error is possible, it should return  <strong>NULL</strong>.




All constant definitions, data type and function declarations (prototypes) must be located in a header file named <strong><em>buffer.h</em></strong>. You are allowed to use only named constants in your programs (except when incrementing something by 1 or setting a numeric value to 0). To name a constant you must use <strong><em>#define</em></strong> preprocessor directive (see <strong><em>buffer.h</em></strong>). The incomplete <strong><em>buffer.h</em></strong> is posted on Brightspace (BS). The function definitions must be stored in a file named <strong><em>buffer.c</em></strong>.

<strong> </strong>

<h1><u>Task 2:</u> Testing the Buffer</h1>




To test your program you are to use the test harness program <strong><em>platy_bt.c</em></strong> (do not modify it) and the input files <strong><em>ass1e.pls </em></strong>(an empty file), and <strong><em>ass1.pls</em></strong>. The corresponding output files are <strong><em>ass1e.out</em></strong> and <strong><em>ass1ai.out </em></strong>(mode = 1), <strong><em>ass1mi.out </em></strong>(mode = -1), <strong><em>ass1fi.out </em></strong>(mode =  0). Those files are available as part of the assignment postings. You must create a standard console project named <strong><em>buffer</em></strong> with an executable target <strong><em>buffer</em></strong> (see <strong><em>Creating_C_Project</em></strong> document in Lab0). The project must contain only one header file (<strong><em>buffer.h</em></strong>) and two souce files: <strong><em>buffer.c</em></strong> and <strong><em>platy_bt.c</em></strong>.




Here is a brief description of the program that is provided for you on Brightspace (BS). It simulates “normal” operating conditions for your buffer utility. The program (<strong><em>platy_bt.c</em></strong>) main function takes to parameters from the command line: an input file name and a character (<strong>f</strong> – fixed-size, <strong>a</strong> – additive self-increment, or <strong>m</strong> – multiplicative self increment) specifying the buffer operational mode. It opens up a file with the specified name (for example, <strong><em>ass1.pls</em></strong>), creates a buffer, and loads it with data from the file using the <strong><em>b_load() </em></strong>function. Then the program prints the current capacity, the current limit, the current operational mode, the increment factor, the current mark, and the contents of the buffer. It packs the buffer, and if the pack operation is successful, it prints the buffer contents again. Your program must not overflow any buffers in any operational mode, no matter how long the input file is. The provided main program will not test all your functions. You are strongly encouraged to test all your buffer functions with your own test files and modified main function.




<h1><u>Bonus Task</u>: Implementing a Preprocessor Macro Definition and Expansion (1%)</h1>

<strong> </strong>

Implement <strong><em>b_isfull()</em></strong> both as a function and a macro expansion (macro). Using conditional processing you must allow the user to choose between using the macro or the function in the compiled code. If <strong>B_FULL</strong> name is defined the macro should be used in the compiled code. If the <strong>B_FULL</strong> name is not defined or undefined, the function should be used in the compiled code.

To receive credit for the bonus task your code must be well documented, tested, and working.




<strong><em>SUBMIT THE FOLLOWING: </em></strong>




<strong><u>Paper Submission:</u> Hand</strong> in on paper, the fully documented source code of your <strong><em>buffer.h</em></strong> and <strong><em>buffer.c</em></strong> files. Print your output for the test files <strong><em>ass1.pls </em></strong>and <strong><em>ass1e.pls</em></strong>. Include a description or listing of your own test file(s) (output and/or input, if appropriate) showing how you tested your program.  Don’t kill a hundred trees; submit descriptions and short excerpts of your testing inputs and outputs if the files are large. Save the Third Rock from the Sun! <strong><em>Print and submit the Marking Sheet for Assignment 1 with your Name and Student ID filled in.</em></strong><strong><em><sub>  </sub></em></strong>

<strong> </strong>

<strong><u>Digital Submission</u>: Compress</strong> into a <strong>zip</strong> file the following files<em>: platy_bt.c, buffer.h, buffer.c, ass1.pls, ass1e.pls </em>and the corresponding<em> test output files </em>produced by your program. Include your additional input/output test files if you have any. Upload the zip file on Brightspace. The file must be submitted prior or on the due date as indicated in the assignment. The name of the file must be <strong><u>Your Last Name</u></strong> followed by the last three digits of your student number. For example: Ranev345.zip.




Make sure all printed materials are placed into an unsealed envelope and are deposited into my assignment submission box prior to the end of the due date. If the due time is midnight, you can make the submission the next morning. The submission must follow the course submission standards. You will find the Assignment Submission Standard as well as the Assignment Marking Guide (<strong>CST8152_ASSAMG.pdf)</strong> for the Compilers course on the Brightspace.




<strong>Assignments will not be marked if the source files are not submitted on time.</strong> Assignments could be late, but the lateness will affect negatively your mark: see the Course Outline and the Marking Guide. All assignments must be successfully completed to receive credit for the course, even if the assignments are late.




<strong>Evaluation Note:</strong> Make your functions as efficient as possible. These functions are called many times during the compilation process. The functions will be graded with respect to design, documentation, error checking, robustness, and efficiency.  When evaluating and marking your assignment, I will use the standard project and <strong><em>platy_bt.c</em></strong> and the test files posted on the net. If your program compiles, runs, and produces correct output files, it will be considered a <em>working program</em>. Additionally, I will try my best to “crash” your functions using a modified main program, which will test all your functions including calling them with “invalid” parameters. I will use also some additional test files (for example, a large file). This can lead to fairly big reduction of your assignment mark (see<strong> CST8152_ASSAMG </strong>and <strong><em>c</em>MarkingSheetA1</strong> documents).




Enjoy the assignment. And do not forget that:

<h2> “Writing a program is like painting. It is better to start on a new canvas.” Ancient  P-Artist “It is part of the nature of humans to begin with romance (buffer) and build to reality</h2>

<strong><em>(compiler).”  </em></strong>by Ray Bradbury<strong><em>                                                                                                               </em></strong>

<strong>#define buff·er </strong>(bùf¹er) <em>noun (Microsoft Bookshelf)</em>

<ol>

 <li>Something that lessens or absorbs the shock of an impact.</li>

 <li>One that protects by intercepting or moderating adverse pressures or influences: <em>“A sense of humor . . . may have served as a buffer against the . . . shocks of disappointment”</em> (James Russell Lowell).</li>

 <li>Something that separates potentially antagonistic entities, as an area between two rival powers that serves to lessen the danger of conflict.</li>

 <li><em>Chemistry</em>. A substance that minimizes change in the acidity of a solution when an acid or base is added to the solution.</li>

 <li><strong><em>Computer Science</em></strong><strong>. A device or memory area used to store data temporarily and deliver it at a rate different from that at which it was received.</strong></li>

</ol>
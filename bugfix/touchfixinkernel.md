Touch fixation
Get the memory address of touch panel drivers in the kernel
"echo 0 > /proc/sys/kernel/kptr_restrict
Otherwise the next command on kallsyms file will return 00000000
cat /proc/kallsyms > /sdcard/symbol.txt
cat /proc/kallsyms | grep tpd_i2c_probe
Return is c092f968 t tpd_i2c_probe
This implicates the touch driver is at the aforementioned address in the error free decompressed kernel disassembled in ida pro.

Kernel decompression

The split_img folder contains the compressed kernel as recovery.img-zImage file. Copy the file to the storage. As its epitomized utilizing gunzip compression algorithm, rename the file appending the extension gz. But there are some extra binary data as the header and footer of the file, making the file undecompressable. So the archive file must be refined with hex edition.

Hex editor app https://play.google.com/store/apps/details?id=tuba.tools
Header removal
Gunzip files start with a hex file signature "1f8b". Search the hex value and cut all the hex values before the file signature. Save the cut hex value to a new backup file as header for future annexation.
Footer removal
Search hex value "6D6564696174656B" which corresponds to text value "mediatek". Scroll up and find text value "..PP..............." with a total of 15 dots which may be sundry across different kernels. For instances primo n2 twrp kernel has text value "Ntc................." with 17 dots. The last dot with hex value of 01 and NOT the usual 00 implicates the end of gunzip file. The dot may have sundry hex values such as 01 in primo h8 and e0 a0 9c 62 in primo n2. LEAVE the hex value corresponding to the FIRST DOT which is "..PP." or hex "1d 00 50 50 01" for primo h8 and hex values corresponding to the first TWO DOTS which are "Ntc.." or hex "4e 54 63 e0 00" for primo n2. So sometimes the exact porton to cut must be determined with TRIAL AND ERROR method. Cut the rest of the hex values containing 15-1=14 dots for h8 and 17-2=15 dots for n2 and numerous characters. Save the cut hex values in a new backup file footer.

Save the changes made directly to the compressed file. DO NOT choose save as option as its notorious to append black hex values comprising of numerous dots ".........." at the end of the file, resulting in corrupted gunzip file and wrong file content size demonstration. Check the archive file info in 7zip software and verify that the content is demonstrated with appropriate decompressed file size.


Rebuild
Open modified kernel file. Open header file and copy all values and paste the data at the beginning of the kernel file. Choose SAVE AS this time, otherwise the file data will be jumbled. Then open the saved as new file and check if the last offset 006ab5af of the saved as file matches the last offset 006af7fd of the modified kernel with header. If there's a mismatch, cut the missed data from the last offset to the end of the modified kernel 006ab5af to 006af7fd and paste the data at the end of the saved as file.

The cut data is exactly 424e values, which implicates pasting data at the beginning can't increase file size, instead removes data from the last portion accordingly. So the best practice is to paste data at the end of file.


Header in kernel is working
Paste header at the kernel start, must save as and paste missed.
Pasted missed data 006ab5af to 006af7fd can be saved directly
Paste footer at the end of header plus kernel, direct save works
Final offset should be at 006bfd3f implying text sunwave_fp.

kernel in header
Pasting kernel data at the end of the header and saving directly results in data loss with supplantation by numerous dots. Choosing save as results in only the header being saved again, rest is lost.

▀End hex values and offsets size of each part
Important during reannexation after touch sensor fixation.
header 0000424e or 16974 bytes implying text "error."
unmodified kernel 006ab5ae or 6993326 bytes implying text "PP."
footer 00010541 or 66881 implying text "sunwave_fp."


▀Hex editor fixed stock and final kernel offsets
Kernel start both 0000424f
Footer start stock 006af7fe final 006ae75c
Footer or file end stock 006bfd3f final 006bec9d or 7072925 bytes

Ida pro
Processor arm little endian
Rom start address 0xC0008000
Loading start address 0xC0008000
Choose no which implies 32 bit mode
Wait for the disassembler till the green circle appears at the top
File then script file then kallsyms_loader.idc then symbols.txt
Wait for the completion of disassembly in 15 minutes
Options then general disassembly set number of opcode bytes to 6
Right click on yellow rom addresses and jump to address c092f968
Find out assembly language command bl cmp and mov or 
Edit then patch program then change bytes
Supplant bl with nop 00 F0 20 E3
Supplant cmp with nop 00 F0 20 E3
Supplant mov with nop 00 F0 20 E3
Or supplant B.EQ with nop 00 F0 20 E3 according to xda doogee
Edit then patch program then apply patches to input file

and I replaced statement and return with NOP (NOP for 64bit ARM its 1F 20 03 D5 and for 32bit ARM its 00 F0 20 E3).


Hex editor app patch
Copy hex from cmp command up to 5 lines totalling 20 hex values
The hex values are supposedly 020050E37700000A5432
Search the hex values in the app and verify with hex comparison
Supersede the first hex 02 to 00 implying normal boot and save

Rename the file to recovery.img-zimage
Open error free gzip kernel and examine first few values
First two values 1f8b is the gzip file signature
Third value 08 indicates deflate compression method
Ninth value 02 denotes compression type as maximum
Compress with the command gzip -n -k -9
Paste header in kernel followed by footer paste at the end


File pointer will change if the size of the kernel changes. Gzip file size mismatch is acceptable as per github instructions and pdf tutorial 6243 to 6271 kilobytes. But practical experience connotes otherwise.


Kernel recompression

Installed magisk module "cross compiled binaries" by zackptg5 and installed original gzip v1.10 via terminal command "ccbins". After two reboots, command "gzip -h" demonstrated the flag "-n or --no--name". Recompression of the recovery.img-zImage_hexfix kernel file with command "gzip -n -k -9" resulted in compressed gzip kernel of size 6993326 which is 1 byte less than size of hexnotfix_errorfree gzip kernel. This nuanced size mismatch was fixed by appending 10 lines of eight "00" bytes or 80 bytes at the end of recovery.img-zImage_hexfix kernel which increased the file size from 22040576 to 22040656. This time the recompression created a gzip size of 6993327 bytes matching exactly with hexnotfix_errorfree gzip kernel. After appending header and footer data the final recovery.img-zImage kernel file reached a size of 7077184 that matched with the stock kernel which had disabled touch.
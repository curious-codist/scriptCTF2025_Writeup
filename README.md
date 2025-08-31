# Challenge 1 - ForeignDesign(Rev)

In this challenge, we are given a .jar file. On inspecting it, I found out that it contains two Java class files(Main and NativeLoader). I also found the existence of a native shared library(.so file). 
## Step 1 - Decompiling class files
After opening Main.class and using jd-gui to decompile the file, I found out that it calls loadLibrary() from NativeLoader
```
public class Main {
  public static void main(String[] args) {
    try {
      NativeLoader.loadLibrary();
    } catch (Exception e) {
      e.printStackTrace();
    } 
  }
}
```
This in-turn, just loads the shared library, based on the operating system. 

## Step 2 - Decompiling the native library
Using Ghidra to decompile linux64/libforeign.so, I found out the JNI-exported functions ava_xyz_scriptctf_Main_initialize()
```
void Java_xyz_scriptctf_Main_initialize(long *param_1,undefined8 param_2){
}
```
It contains mostly JNI function pointers(like FindClass, GetMethodID). The importtant role that it plays is it creates two arrays 
```
int ll[] = {32, 92, 4, 104, 106, 76, 96, 113, 42, 65, 22, 43, 203, 84, 220, 98, 210, 71, 29, 123, 20, 125, 199};
int lll[] = {76, 230, 117, 243, 84, 54, 103, 197, 104, 251, 83, 253, 128, 159};
```
Now, in the function Java_xyz_scriptctf_Main_sc(),
```
void Java_xyz_scriptctf_Main_sc(long *param_1,undefined8 para m_2,undefined2 param_3,uint param_4){
    piVar9 = (int *)(lVar6 + -0x5c + (ulong)param_4 * 4);
    if ((int)param_4 < 0x17) {
        piVar9 = (int *)(lVar5 + (long)(int)param_4 * 4);
    }
    iVar1 = *piVar9;
    if (((long)(int)param_4 & 1U) == 0) {
        iVar2 = FUN_001013a0(param_3,param_4);
    }
    else {
        lVar7 = (**(code **)(*param_1 + 0x388))(param_1,param_2, &DAT_00102070,"(CI)I");
        if (lVar7 == 0) goto LAB_001015ba;
        iVar2 = (**(code **)(*param_1 + 0x408))(param_1,param_2,l Var7,param_3,param_4);
    }
}
```
## Step 3 - Understanding the logic
Basically, it defines an array piVar9, so it takes 0x17(23) elements from array ll and afterwards, takes 14 elements from lll. Hence 
```iVar1=ll+lll```
If the index is even, it calls the function FUN_001012a0 to it. Otherwise, it calls the s2 function from Main.class
```
uint FUN_001013a0(uint param_1,int param_2){
    return param_2 * 3 + (param_2 + 0x13U ^ param_1) ^ 0x5a;
}
```
We can reverse this function, as c = ((v - i*3) ^ 0x5a) ^ (i+0x13)
Also, for the function s2
```
public static int s2(char c, int i) {
    int base = c + i % 7 * 2;
    if (i % 2 == 0){
    base ^= 0x2C;
    } else{
        base ^= 0x13;
    }
    return base + (i & 0x1);
}
```
As the index is always odd, we can reverse this as well, ((val - (i & 1)) ^ 0x13) - 2*(i % 7)

## Step 4 - Python script for Flag
Overall, I made a Python script for the flag - 

```
ll = [32, 92, 4, 104, 106, 76, 96, 113, 42, 65, 22, 43, 203, 84, 220, 98, 210, 71, 29, 123, 20, 125, 199]
lll = [76, 230, 117, 243, 84, 54, 103, 197, 104, 251, 83, 253, 128, 159]
arr = ll + lll
N=len(arr)
def i_even(val, i):
    X = (val ^ 0x5a) - i*3
    c = X ^ (i + 0x13)
    return c & 0xFFFF

def i_odd(val, i):
    y = val - (i & 1)
    b = y ^ 0x13
    c = b - 2*(i % 7)
    return c & 0xFFFF

ws = [0] * N
for i, e in enumerate(arr):
    idx = (i * 5 + 3) % N
    if i % 2 == 0:
        ch_val = i_even(e, i)
    else:
        ch_val = i_odd(e, i)
    ch_byte = ch_val & 0xFF
    ws[idx] = chr(ch_byte)

flag = ''.join(ws)
print("Flag:", flag)
```
## Flag - scriptCTF{nO_MOr3_n471v3_tr4N5l471on}

# Challenge 2 - Secure Server(Crypto)

In this challenge, we are given a capture.pcap file and the server.py. After inspecting the server file, we see that the server first asks for the secret XOR'ed with our key(call it key1). That means the first input is **flag XOR key1**, which we know.  
Now, the server generates a random key(call it key2) and XOR's the input with key2. Hence, **enc2 = flag XOR key1 XOR key2**, which they have printed for us.  
Finally, it takes **enc2 XOR key1** as an input, which is equal to **flag XOR key1 XOR key2 XOR key1** = **flag XOR key2**, which we also know.  
So, we know:
- flag XOR key1 = 151e71ce4addf692d5bac83bb87911a20c39b71da3fa5e7ff05a2b2b0a83ba03
- flag XOR key1 XOR key2 = e1930164280e44386b389f7e3bc02b707188ea70d9617e3ced989f15d8a10d70
- flag XOR key2 = 87ee02c312a7f1fef8f92f75f1e60ba122df321925e8132068b0871ff303960e

Based on this information, if we were take the XOR of all three values, we would get **flag XOR flag XOR flag XOR key1 XOR key XOR key2 XOR key2**, which is same as **flag**.

Taking the XOR, we get 7363726970744354467b7830725f31735f6e6f745f733363757233212121217d, which is the flag. Converting to ascii gives us the flag
## Flag - scriptCTF{x0r_1s_not_s3cur3!!!!}

# Challenge 3 - pdf(Forensics)

In this challenge, we are given a .pdf file. Running file on it tells us that it, indeed is a pdf file with a single page.  
<img width="748" height="65" alt="image" src="https://github.com/user-attachments/assets/39faf1e9-1e03-4948-b114-44b3e7c3e9df" />

Running binwalk to check for hidden files, we get:  

<img width="1875" height="222" alt="image" src="https://github.com/user-attachments/assets/66b89c54-6b11-44ef-8cca-314ab3f865dc" />


We can see that there is something hidden in the pdf file. Extracting the hidden files, we get a file called 11B. When we cat 11B, we get the flag.  


<img width="750" height="114" alt="image" src="https://github.com/user-attachments/assets/39911eea-a770-4aa5-b72d-a75d7de1ccd1" />  

## Flag - scriptCTF{pdf_s7r34m5_0v3r_7w17ch_5tr34ms}


<div align="center">

## Playing Audio in Windows using waveOut Interface


</div>

### Description

This tutorial will teach you how to use the Windows waveOut multimedia functions. It also explains a little about how audio is stored in the digital form. I hope this tutorial is useful. Full source code is included as is a downloadable version wrapped in an MSVC++ project.
 
### More Info
 


<span>             |<span>
---                |---
**Submitted On**   |
**By**             |[David Overton](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByAuthor/david-overton.md)
**Level**          |Intermediate
**User Rating**    |5.0 (139 globes from 28 users)
**Compatibility**  |C, Microsoft Visual C\+\+
**Category**       |[Graphics/ Sound](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByCategory/graphics-sound__3-15.md)
**World**          |[C / C\+\+](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByWorld/c-c.md)
**Archive File**   |[](https://github.com/Planet-Source-Code/david-overton-playing-audio-in-windows-using-waveout-interface__3-4422/archive/master.zip)





### Source Code

<p>
  <font face="tahoma" size="3">
  <b>
   Windows waveOut Tutorial
  </b>
  </font>
 </p>
 <p>
  <font face="tahoma" size="2">
  This tutorial is designed to help you use the windows waveOut interface
  for playing digital audio. I know from experience that the interface can be
  pretty difficult to get to grips with. Through this tutorial I will build
  a windows console application for playing raw digital audio. This is my first
  tutorial so I'll apologise for the mistakes in advance!
  <br><br>
  Note: This tutorial assumes that you are competent with C programming and
  using the Windows API functions. A basic understanding of digital audio is
  useful but not completely necessary.
  <br>
  <br>
  <b>Contents</b>
  <ul>
   <li>Get The Documentation!</li>
   <li>What is Digital Audio?</li>
   <li>Opening the Sound Device</li>
   <li>Playing a Sound</li>
   <li>Streaming Audio to the Device</li>
   <li>The Buffering Scheme</li>
   <li>The Driver Program</li>
   <li>What Next?</li>
   <li>Contacting Me</li>
  </ul>
  </font>
 </p>
 <p>
  <font face="tahoma" size="2">
  <b>
   Get The Documentation!
  </b>
  </font>
 </p>
 <p>
  <font face="tahoma" size="2">
  The first thing you'll need is some decent documentation on the waveOut
  interface. If you have the Microsoft Platform SDK (a worthwhile download)
  or a copy of Visual C++ then you already have the relevent information in
  the documentation provided. If you don't have either of these you can
  view the documentation online at Microsoft's Developer website (msdn.microsoft.com).
  </font>
 </p>
 <p>
  <font face="tahoma" size="2">
  <b>
   What is Digital Audio?
  </b>
  </font>
 </p>
 <p>
  <font face="tahoma" size="2">
  This bit is for people who have absolutely no idea how digital audio is stored. Skip this
  section if you know all about digital audio and you know the meaning of the terms 'Sample',
  'Sampling Rate', 'Sample Size', and 'Channels'.
  <br>
  <br>
  It's all very well sending all these bytes to the sound card but what do these bytes mean?
  Audio is simply a series of moving pressure waves. In real life this is an analogue
  wave, but in the digital world we have to store it as a set of samples along this
  wave. A sample is a value that represents the amplitude of the wave at a given point in time
  - it's just a number.
  <br>
  <br>
  The sampling rate is how frequently we take a sample of the wave. It is measured in hertz (Hz)
  or 'samples per second'. Obviously the higher the sampling rate, the more like the analogue
  wave your sampled wave becomes, so the higher the quality of the sound.<br>
  <br>
  Another thing that contributes to the quality of the audio is the size of each sample.
  Yes, you guessed it. The larger the sample size the higher the quality of the audio.
  Sample size is measured in bits. Why is the quality better? Consider an 8 bit sample. It has
  256 (2^8) possible values. This means that you may not be able to represent the exact
  amplitude of the wave with it. Now consider a 16 bit sample. It has 65536 possible values
  (2^16). This means that it is 256 times as accurate as the 8 bit sample and can thus
  represent the amplitude more accurately.
  <br>
  <br>
  The final thing I'll touch on here is the channels. On most systems you have two speakers,
  left and right. That's two channels. This means that you must store a sample for the left
  channel and the right channel.<br>
  Fortunately this is easy for two channels (which is the most you'll encounter in this tutorial).
  The samples are interleaved. That is the samples are stored, left, right, left, right etc...
  <br>
  <br>
  CD quality audio is sampled at 44100 Hz, has a sample size of 16 bits and has 2 channels. This
  means that 1 MB of audio data lasts for approximately 6 seconds.
  </font>
 </p>
 <p>
  <font face="tahoma" size="2">
  <b>
   Opening the Sound Device
  </b>
  </font>
 </p>
 <p>
  <font face="tahoma" size="2">
  To open the sound device you use the <b>waveOutOpen</b> function (look this up in your
  documentation now). Like most Windows objects, you basically need a handle to anything
  to use it. When you act on a window you use a HWND handle. Similarly when you act on
  a waveOut device you use a HWAVEOUT handle.
  <br>
  <br>
  So now comes the first version of our application. This simply opens the wave device
  to a CD quality standard, reports what's happened and closes it again.
  <br>
  <br>
  </font>
  <table bgcolor="#e0e0e0">
  <tr>
   <td>
   <pre>
#include &lt;windows.h&gt;
#include &lt;mmsystem.h&gt;
#include &lt;stdio.h&gt;
int main(int argc, char* argv[])
{
  HWAVEOUT hWaveOut; /* device handle */
  WAVEFORMATEX wfx; /* look this up in your documentation */
  MMRESULT result;  /* for waveOut return values */
  /*
   * first we need to set up the WAVEFORMATEX structure.
   * the structure describes the format of the audio.
   */
  wfx.nSamplesPerSec = 44100; /* sample rate */
  wfx.wBitsPerSample = 16;   /* sample size */
  wfx.nChannels   = 2;   /* channels  */
  /*
   * WAVEFORMATEX also has other fields which need filling.
   * as long as the three fields above are filled this should
   * work for any PCM (pulse code modulation) format.
   */
  wfx.cbSize     = 0; /* size of _extra_ info */
  wfx.wFormatTag   = WAVE_FORMAT_PCM;
  wfx.nBlockAlign   = (wfx.wBitsPerSample >> 3) * wfx.nChannels;
  wfx.nAvgBytesPerSec = wfx.nBlockAlign * wfx.nSamplesPerSec;
  /*
   * try to open the default wave device. WAVE_MAPPER is
   * a constant defined in mmsystem.h, it always points to the
   * default wave device on the system (some people have 2 or
   * more sound cards).
   */
  if(waveOutOpen(
    &hWaveOut,
    WAVE_MAPPER,
    &wfx,
    0,
    0,
    CALLBACK_NULL
  ) != MMSYSERR_NOERROR) {
    fprintf(stderr, "unable to open WAVE_MAPPER device\n");
    ExitProcess(1);
  }
  /*
   * device is now open so print the success message
   * and then close the device again.
   */
  printf("The Wave Mapper device was opened successfully!\n");
  waveOutClose(hWaveOut);
  return 0;
}
   </pre>
   </td>
  </tr>
  </table>
  <br>
  <font face="tahoma" size="2">
  Note that when compiling this program you will need to add winmm.lib to
  your list of library files or the linker will fail.
  <br>
  <br>
  So that was the first step. The device was ready and waiting for you to write
  audio data to it.
  </font>
 </p>
 <p>
  <font face="tahoma" size="2">
  <b>
   Playing a Sound
  </b>
  </font>
 </p>
 <p>
  <font face="tahoma" size="2">
  Opening and closing the device is fun for a while but it doesn't actually
  do that much. What we want is to hear a sound. We need to do two things before
  this can happen.
  <ul>
   <li>Obtain a source of raw audio in the correct format</li>
   <li>Work out how to write the data</li>
  </ul>
  Problem 1 is easy to solve. You can convert any music file into raw audio using a
  program like Winamp with the Disk Writer plug-in. Start small and convert one of
  the Windows sounds into a raw file. These files are located in your \Windows\Media
  directory. Ding.wav seems like a good choice to start with. If you can't convert
  this to a raw file you can have fun playing the unconverted file back instead. It
  will sound too fast since these files are mostly sampled at 22 kHz.
  <br>
  <br>
  Problem 2 is slightly more tricky. Audio is written in blocks, each with its own
  header. It's easy to write one block but at some point we're going to have to come
  up with a scheme for queuing and writing many blocks. The reason I said to start
  with a small file is that the second version of our application will load the entire
  file as a single block.
  <br>
  <br>
  We will first tackle Problem 2 by writing a function that will send a block of data
  to the audio device. The function will be called <b>writeAudioBlock</b>. To write
  audio data you use up to three functions. These are <b>waveOutPrepareHeader</b>,
  <b>waveOutWrite</b>, and <b>waveOutUnprepareHeader</b> and are called in the
  order I have listed them. It would be a good idea to look these up in your documentation
  now to familiarise yourself with them.
  <br>
  <br>
  Here is the code for a preliminary version of the function <b>writeAudioBlock</b>
  <br>
  <br>
  </font>
  <table bgcolor="#e0e0e0">
  <tr>
   <td>
   <pre>
void writeAudioBlock(HWAVEOUT hWaveOut, LPSTR block, DWORD size)
{
  WAVEHDR header;
  /*
   * initialise the block header with the size
   * and pointer.
   */
  ZeroMemory(&header, sizeof(WAVEHDR));
  header.dwBufferLength = size;
  header.lpData     = block;
  /*
   * prepare the block for playback
   */
  waveOutPrepareHeader(hWaveOut, &header, sizeof(WAVEHDR));
  /*
   * write the block to the device. waveOutWrite returns immediately
   * unless a synchronous driver is used (not often).
   */
  waveOutWrite(hWaveOut, &header, sizeof(WAVEHDR));
  /*
   * wait a while for the block to play then start trying
   * to unprepare the header. this will fail until the block has
   * played.
   */
  Sleep(500);
  while(waveOutUnprepareHeader(
    hWaveOut,
    &header,
    sizeof(WAVEHDR)
  ) == WAVERR_STILLPLAYING)
    Sleep(100);
}
   </pre>
   </td>
  </tr>
  </table>
  <br>
  <font face="tahoma" size="2">
  Now we've got a function for writing a block of data we need a function for getting
  hold of one in the first place. That is the task of <b>loadAudioBlock</b>. <b>loadAudioBlock</b>
  will load a file into memory and return a pointer to it. Here is the code for <b>loadAudioBlock</b>.
  <br>
  <br>
  </font>
  <table bgcolor="#e0e0e0">
  <tr>
   <td>
   <pre>
LPSTR loadAudioBlock(const char* filename, DWORD* blockSize)
{
  HANDLE hFile  = INVALID_HANDLE_VALUE;
  DWORD size   = 0;
  DWORD readBytes = 0;
  void* block   = NULL;
  /*
   * open the file
   */
  if((hFile = CreateFile(
    filename,
    GENERIC_READ,
    FILE_SHARE_READ,
    NULL,
    OPEN_EXISTING,
    0,
    NULL
  )) == INVALID_HANDLE_VALUE)
    return NULL;
  /*
   * get it's size, allocate memory and read the file
   * into memory. don't use this on large files!
   */
  do {
    if((size = GetFileSize(hFile, NULL)) == 0)
      break;
    if((block = HeapAlloc(GetProcessHeap(), 0, size)) == NULL)
      break;
    ReadFile(hFile, block, size, &readBytes, NULL);
  } while(0);
  CloseHandle(hFile);
  *blockSize = size;
  return (LPSTR)block;
}
   </pre>
   </td>
  </tr>
  </table>
  <br>
  <font face="tahoma" size="2">
  Finally for this section, here are the changes that must be made to
  the beginning of the file and to <b>main</b>.
  <br>
  <br>
  </font>
  <table bgcolor="#e0e0e0">
  <tr>
   <td>
   <pre>
#include &lt;windows.h&gt;
#include &lt;mmsystem.h&gt;
#include &lt;stdio.h&gt;
LPSTR loadAudioBlock(const char* filename, DWORD* blockSize);
void writeAudioBlock(HWAVEOUT hWaveOut, LPSTR block, DWORD size);
int main(int argc, char* argv[])
{
  HWAVEOUT hWaveOut;
  WAVEFORMATEX wfx;
  LPSTR block;    /* pointer to the block */
  DWORD blockSize;  /* holds the size of the block */
        .
        .   (leave middle section as it was)
        .
  printf("The Wave Mapper device was opened successfully!\n");
  /*
   * load and play the block of audio
   */
  if((block = loadAudioBlock("c:\\temp\\ding.raw", &blockSize)) == NULL) {
    fprintf(stderr, "Unable to load file\n");
    ExitProcess(1);
  }
  writeAudioBlock(hWaveOut, block, blockSize);
  waveOutClose(hWaveOut);
  return 0;
}
   </pre>
   </td>
  </tr>
  </table>
  <br>
  <font face="tahoma" size="2">
  If you've put all the code in the correct place and it compiled it will
  now play small audio files. We've accomplished something similar to what
  the <b>PlaySound</b> function does. Try playing with this. Change the playback
  sample rate (in <b>main</b>) or the sample size (multiple of 8 btw) and see what happens,
  or even the number of channels. You'll find that changing the sample rate or
  number of channels speeds up or slows down the audio. Changing the sample size
  has a somewhat devastating affect!
  </font>
 </p>
 <p>
  <font face="tahoma" size="2">
  <b>
   Streaming Audio to the Device
  </b>
  </font>
 </p>
 <p>
  <font face="tahoma" size="2">
  As you can probably see the above code has a number of fundamental flaws
  (note that this was deliberate :), the most evident of which are:
  <ul>
   <li>
   We can't play very large files due to the way they are loaded. The current
   method buffers the entire file and plays it all back at once. Audio by its
   very nature is large so we need to find a way of streaming the data to the
   device block by block.
   </li>
   <li>
   The current version of writeAudioBlock is synchronous so writing multiple
   blocks bit by bit will cause a gap between each block output (we can't refill
   the buffer fast enough). Microsoft recommends at least a double buffering
   scheme so that you fill one block while another is playing and then switch the
   blocks. This itself is not nearly enough. Even switching the blocks will cause
   a very small (but annoying) gap in the output.
   </li>
  </ul>
  Fortunately reading in blocks is a very easy exercise so I will defer from writing
  the code for that right now. Rather, I will concentrate on a buffering scheme for
  writing audio to the device in a gapless stream.
  <br>
  <br>
  This problem of block switching is not nearly as serious as it sounds. No you can't
  switch two blocks without a gap but the interface does something which allows you
  to get around this. It maintains a queue of blocks. Any block which you have passed
  through the <b>waveOutPrepareHeader</b> function can be inserted into the queue
  using <b>waveOutWrite</b>. This means we can write 2 (or more) blocks to the device
  and fill a third while the first is playing, then perform the switch while the second
  is playing. This gives us gapless output.
  <br>
  <br>
  The final problem before I describe a method of doing this is, how do we know when a
  block has finished playing? I was doing something very bad in the first version of
  <b>writeAudioBlock</b> and polling the device using <b>waveOutUnprepareHeader</b> until
  the block had finished. We can't do this any more because we need the time to refill
  audio blocks, and there are much better ways offered by the waveOut interface.
  <br>
  <br>
  The waveOut interface offers 4 types of callback mechanism to notify you of when blocks
  have finished playing. These are:
  <ul>
   <li>An event - an event is set when a block completes</li>
   <li>A callback function - a function is called when a block completes</li>
   <li>A thread - a thread message is sent when a block completes</li>
   <li>A window - a window message is sent when a block completes</li>
  </ul>
  The way you specify which of these is used is in the dwCallback parameter of the
  <b>waveOutOpen</b> function. In my method we will be using a function as the callback.
  <br>
  <br>
  So we need a new function: <b>waveOutProc</b>. This (user defined) function is actually
  documented so you can look that up now. As you can see the function is called for three
  things: When the device is opened, closed, and when a block finishes. We are only interested
  in the call for when a block finishes.
  </font>
 </p>
 <p>
  <font face="tahoma" size="2">
  <b>
   The Buffering Scheme
  </b>
  </font>
 </p>
 <p>
  <font face="tahoma" size="2">
  My buffering scheme works on a principle similar to that discussed above. It requires the
  use of a variable that keeps count of the number of free buffers at any time (yes a semaphore
  would be ideal here but we can't use one, I'll explain why later). This variable is initialised
  to the number of blocks, decremented when a block is written and incremented when a block
  completes. When no blocks are available we wait until the counter is at least 1 and then
  continue writing. This allows us to queue any number of blocks in a ring which is very effective.
  Rather than queuing 3 blocks, I queue more like 20, of about 8 kB each.
  <br>
  <br>
  Now here's something you might have already guessed: <b>waveOutProc</b> is called from a
  different thread. Windows create a thread specifically for managing the audio playback.
  There are a number of restrictions on what you can do in this callback. To quote the Microsoft
  Documentation:
  </font>
  <pre>
 &quot;Applications should not call any system-defined functions from
  inside a callback function, except for EnterCriticalSection,
  LeaveCriticalSection, midiOutLongMsg, midiOutShortMsg,
  OutputDebugString, PostMessage, PostThreadMessage, SetEvent,
  timeGetSystemTime, timeGetTime, timeKillEvent, and timeSetEvent.
  Calling other wave functions will cause deadlock.&quot;
  </pre>
  <font face="tahoma" size="2">
  Which explains why we can't use a semaphore - it would require the use of <b>ReleaseSemaphore</b>
  which you shouldn't use. In practice it is a little more flexible than this - I have seen
  code that uses semaphores from the callback but what works on one Windows version may not
  work on another. Also, calling waveOut functions from the callback does cause deadlock.
  Ideally we would also call <b>waveOutUnprepareHeader</b> in the callback but we can't do
  that (it doesn't deadlock until you call <b>waveOutReset</b> just for your information :)
  <br>
  <br>
  You'll notice that <b>waveOutOpen</b> provides a method of passing instance data
  (a user defined pointer) to the callback function. We're going to use this to pass a pointer
  to our counter variable.
  <br>
  <br>
  One more thing before we write the <b>waveOutProc</b> function by the way. Since <b>waveOutProc</b>
  is called from a different thread, two threads will end up writing to the block counter variable.
  To avoid any conflict we need to use a Critical Section object (which will be a static module
  variable called waveCriticalSection).
  <br>
  <br>
  Here is the <b>waveOutProc</b> function:
  <br>
  <br>
  </font>
  <table bgcolor="#e0e0e0">
  <tr>
   <td>
   <pre>
static void CALLBACK waveOutProc(
  HWAVEOUT hWaveOut,
  UINT uMsg,
  DWORD dwInstance,
  DWORD dwParam1,
  DWORD dwParam2
)
{
  /*
   * pointer to free block counter
   */
  int* freeBlockCounter = (int*)dwInstance;
  /*
   * ignore calls that occur due to openining and closing the
   * device.
   */
  if(uMsg != WOM_DONE)
    return;
  EnterCriticalSection(&waveCriticalSection);
  (*freeBlockCounter)++;
  LeaveCriticalSection(&waveCriticalSection);
}
   </pre>
   </td>
  </tr>
  </table>
  <br>
  <font face="tahoma" size="2">
  The next thing we need is a couple of functions for allocating and freeing the block memory
  and a new implementation of <b>writeAudioBlock</b> called <b>writeAudio</b>.
  Here are the functions <b>allocateBlocks</b> and <b>freeBlocks</b>. <b>allocateBlocks</b> allocates
  a set number of blocks, with headers at a given size, and <b>freeBlocks</b> frees this memory.
  <b>allocateBlocks</b> will cause the program to exit if it fails. This means we don't need to
  check its return value in <b>main</b>.
  <br>
  <br>
  </font>
  <table bgcolor="#e0e0e0">
  <tr>
   <td>
   <pre>
WAVEHDR* allocateBlocks(int size, int count)
{
  unsigned char* buffer;
  int i;
  WAVEHDR* blocks;
  DWORD totalBufferSize = (size + sizeof(WAVEHDR)) * count;
  /*
   * allocate memory for the entire set in one go
   */
  if((buffer = HeapAlloc(
    GetProcessHeap(),
    HEAP_ZERO_MEMORY,
    totalBufferSize
  )) == NULL) {
    fprintf(stderr, "Memory allocation error\n");
    ExitProcess(1);
  }
  /*
   * and set up the pointers to each bit
   */
  blocks = (WAVEHDR*)buffer;
  buffer += sizeof(WAVEHDR) * count;
  for(i = 0; i &lt; count; i++) {
    blocks[i].dwBufferLength = size;
    blocks[i].lpData = buffer;
    buffer += size;
  }
  return blocks;
}
void freeBlocks(WAVEHDR* blockArray)
{
  /*
   * and this is why allocateBlocks works the way it does
   */
  HeapFree(GetProcessHeap(), 0, blockArray);
}
   </pre>
   </td>
  </tr>
  </table>
  <br>
  <font face="tahoma" size="2">
  The new function <b>writeAudio</b> needs to queue as many blocks as necessary to write
  the data. The basic algorithm is:
  <br>
  <br>
  </font>
  <table>
  <tr>
   <td>
   <pre>
  While there's data available
    If the current free block is prepared
      Unprepare it
    End If
    If there's space in the current free block
  		Write all the data to the block
        Exit the function
    Else
        Write as much data as is possible to fill the block
        Prepare the block
        Write it
        Decrement the free blocks counter
        Subtract however many bytes were written from the data available
        Wait for at least one block to become free
        Update the current block pointer
    End If
  End While
   </pre>
   </td>
  </tr>
  </table>
  <font face="tahoma" size="2">
  This raises a question: How do I tell when a block is prepared and when it isn't?<br>
  This is a fairly easy one actually. Windows makes use of the dwFlags member of the WAVEHDR
  structure. It is used for a few things but one thing <b>waveOutPrepareHeader</b> does
  is set the WHDR_PREPARED flag. All we have to do is test for the flag in the dwFlags
  member.
  <br>
  <br>
  I will make use of the dwUser member of the WAVEHDR structure to maintain a count of
  how full a block is. Here is the listing for the <b>writeAudio</b> function:
  <br>
  <br>
  </font>
  <table bgcolor="#e0e0e0">
  <tr>
   <td>
   <pre>
void writeAudio(HWAVEOUT hWaveOut, LPSTR data, int size)
{
  WAVEHDR* current;
  int remain;
  current = &waveBlocks[waveCurrentBlock];
  while(size &gt; 0) {
    /*
     * first make sure the header we're going to use is unprepared
     */
    if(current-&gt;dwFlags & WHDR_PREPARED)
      waveOutUnprepareHeader(hWaveOut, current, sizeof(WAVEHDR));
    if(size &lt; (int)(BLOCK_SIZE - current-&gt;dwUser)) {
      memcpy(current-&gt;lpData + current-&gt;dwUser, data, size);
      current-&gt;dwUser += size;
      break;
    }
    remain = BLOCK_SIZE - current-&gt;dwUser;
    memcpy(current-&gt;lpData + current-&gt;dwUser, data, remain);
    size -= remain;
    data += remain;
    current-&gt;dwBufferLength = BLOCK_SIZE;
    waveOutPrepareHeader(hWaveOut, current, sizeof(WAVEHDR));
    waveOutWrite(hWaveOut, current, sizeof(WAVEHDR));
    EnterCriticalSection(&waveCriticalSection);
    waveFreeBlockCount--;
    LeaveCriticalSection(&waveCriticalSection);
    /*
     * wait for a block to become free
     */
    while(!waveFreeBlockCount)
      Sleep(10);
    /*
     * point to the next block
     */
    waveCurrentBlock++;
    waveCurrentBlock %= BLOCK_COUNT;
    current = &waveBlocks[waveCurrentBlock];
    current-&gt;dwUser = 0;
  }
}
   </pre>
   </td>
  </tr>
  </table>
  <br>
  <font face="tahoma" size="2">
  Now we have this new function for writing the audio you can scrap the <b>writeAudioBlock</b>
  function since it's not being used any more. You can also scrap the <b>loadAudioBlock</b>
  function because the next section will start a new implementation of <b>main</b> that doesn't
  require <b>loadAudioBlock</b>.
  </font>
 </p>
 <p>
  <font face="tahoma" size="2">
  <b>
   The Driver Program
  </b>
  </font>
 </p>
 <p>
  <font face="tahoma" size="2">
  If you've followed this tutorial right though you will now have a C file containing
  the following functions:
  <ul>
   <li><b>main</b></li>
   <li><b>waveOutProc</b></li>
   <li><b>allocateBlocks</b></li>
   <li><b>freeBlocks</b></li>
   <li><b>writeAudio</b></li>
  </ul>
  Note that this file won't compile until we strip off the old version of <b>main</b> and
  declare the module variables needed.
  <br><br>
  We're now going to write a completely new version of <b>main</b> that will stream files
  from disk to the waveOut device. This listing also contains the declarations for the module
  variables and the prototypes for the functions we've already written.
  <br>
  <br>
  </font>
  <table bgcolor="#e0e0e0">
  <tr>
   <td>
   <pre>
#include &lt;windows.h&gt;
#include &lt;mmsystem.h&gt;
#include &lt;stdio.h&gt;
/*
 * some good values for block size and count
 */
#define BLOCK_SIZE 8192
#define BLOCK_COUNT 20
/*
 * function prototypes
 */
static void CALLBACK waveOutProc(HWAVEOUT, UINT, DWORD, DWORD, DWORD);
static WAVEHDR* allocateBlocks(int size, int count);
static void freeBlocks(WAVEHDR* blockArray);
static void writeAudio(HWAVEOUT hWaveOut, LPSTR data, int size);
/*
 * module level variables
 */
static CRITICAL_SECTION waveCriticalSection;
static WAVEHDR*     waveBlocks;
static volatile int   waveFreeBlockCount;
static int       waveCurrentBlock;
int main(int argc, char* argv[])
{
  HWAVEOUT hWaveOut; /* device handle */
  HANDLE  hFile;  /* file handle */
  WAVEFORMATEX wfx; /* look this up in your documentation */
  char buffer[1024]; /* intermediate buffer for reading */
  int i;
  /*
   * quick argument check
   */
  if(argc != 2) {
    fprintf(stderr, "usage: %s &lt;filename&gt;\n", argv[0]);
    ExitProcess(1);
  }
  /*
   * initialise the module variables
   */
  waveBlocks     = allocateBlocks(BLOCK_SIZE, BLOCK_COUNT);
  waveFreeBlockCount = BLOCK_COUNT;
  waveCurrentBlock  = 0;
  InitializeCriticalSection(&waveCriticalSection);
  /*
   * try and open the file
   */
  if((hFile = CreateFile(
    argv[1],
    GENERIC_READ,
    FILE_SHARE_READ,
    NULL,
    OPEN_EXISTING,
    0,
    NULL
  )) == INVALID_HANDLE_VALUE) {
    fprintf(stderr, "%s: unable to open file '%s'\n", argv[0], argv[1]);
    ExitProcess(1);
  }
  /*
   * set up the WAVEFORMATEX structure.
   */
  wfx.nSamplesPerSec = 44100; /* sample rate */
  wfx.wBitsPerSample = 16;   /* sample size */
  wfx.nChannels    = 2;   /* channels  */
  wfx.cbSize     = 0;   /* size of _extra_ info */
  wfx.wFormatTag   = WAVE_FORMAT_PCM;
  wfx.nBlockAlign   = (wfx.wBitsPerSample * wfx.nChannels) &gt;&gt; 3;
  wfx.nAvgBytesPerSec = wfx.nBlockAlign * wfx.nSamplesPerSec;
  /*
   * try to open the default wave device. WAVE_MAPPER is
   * a constant defined in mmsystem.h, it always points to the
   * default wave device on the system (some people have 2 or
   * more sound cards).
   */
  if(waveOutOpen(
    &hWaveOut,
    WAVE_MAPPER,
    &wfx,
    (DWORD_PTR)waveOutProc,
    (DWORD_PTR)&waveFreeBlockCount,
    CALLBACK_FUNCTION
  ) != MMSYSERR_NOERROR) {
    fprintf(stderr, "%s: unable to open wave mapper device\n", argv[0]);
    ExitProcess(1);
  }
  /*
   * playback loop
   */
  while(1) {
    DWORD readBytes;
    if(!ReadFile(hFile, buffer, sizeof(buffer), &readBytes, NULL))
      break;
    if(readBytes == 0)
      break;
    if(readBytes &lt; sizeof(buffer)) {
      printf("at end of buffer\n");
      memset(buffer + readBytes, 0, sizeof(buffer) - readBytes);
      printf("after memcpy\n");
    }
    writeAudio(hWaveOut, buffer, sizeof(buffer));
  }
  /*
   * wait for all blocks to complete
   */
  while(waveFreeBlockCount &lt; BLOCK_COUNT)
    Sleep(10);
  /*
   * unprepare any blocks that are still prepared
   */
  for(i = 0; i &lt; waveFreeBlockCount; i++)
    if(waveBlocks[i].dwFlags & WHDR_PREPARED)
      waveOutUnprepareHeader(hWaveOut, &waveBlocks[i], sizeof(WAVEHDR));
  DeleteCriticalSection(&waveCriticalSection);
  freeBlocks(waveBlocks);
  waveOutClose(hWaveOut);
  CloseHandle(hFile);
  return 0;
}
   </pre>
   </td>
  </tr>
  </table>
 </p>
 <p>
  <font face="tahoma" size="2">
  <b>
   What Next?
  </b>
  </font>
 </p>
 <p>
  <font face="tahoma" size="2">
  What you do now is up to you. I have a few possibly entertaining suggestions:
  <ul>
   <li>Try modifying the rawaudio program so that it reads from standard input.
     This would make an application that you can directly pipe audio into from
     the command line.</li>
   <li>Rework the reader so that it reads Wave (*.wav) files as opposed to RAW files.
     You will find this surprisingly easy, wave files contain a WAVEFORMATEX structure
     to describe their format which you can use when opening the device. See
     wotsit's format (http://www.wotsit.org) for information on the wave file format.</li>
   <li>See if you can come up with any new or better buffering schemes</li>
   <li>Try attaching this code to an open source decoder such as the Vorbis decoder
     or an MP3 decoder that you can acquire the source to. You then have your the beginnings
     of your own media player.</li>
  </ul>
  You can see get my example winamp plug-in from <a href="http://www.insomniavisions.com/software">
  http://www.insomniavisions.com/software</a> and try that. It is also open source.
  </font>
 </p>
 <p>
  <font face="tahoma" size="2">
  <b>
   Contacting Me
  </b>
  </font>
 </p>
 <p>
  <font face="tahoma" size="2">
  You can contact me (David Overton) by the usual methods available on Planet Source Code.
  <br><br>
  You can also go to <a href="http://www.insomniavisions.com/feedback">http://www.insomniavisions.com/feedback</a>
  to send feedback from my Website.
  <br><br>
  A complete working example of the code on this page can be downloaded <a href="http://download.insomniavisions.com/sources/rawaudio.zip">here</a>.
</font>
</p>


#ifndef ODFILEBUF_DEFINED
#define ODFILEBUF_DEFINED

#if (defined(WIN32) || defined(WIN64)) && !defined(_WIN32_WCE)

#include "OdStreamBuf.h"
#include "RxObjectImpl.h"
#include "OdString.h"
#include "DbSystemServices.h"

#define DD_UNICODE

class OdBaseFileBuf;

/** Remarks:
  This template class is a specialization of the OdSmartPtr class for OdBaseFileBuf object pointers.
*/
typedef OdSmartPtr<OdBaseFileBuf> OdBaseFileBufPtr;

/** Description:
  This class is the base file I/O class.
  Library:
  Source *code* provided.
  {group:ExServices_Classes}
*/
class OdBaseFileBuf: public OdRxObjectImpl<OdStreamBuf>
{
private:
  bool m_bError;
protected:
  inline void enableErrorMode(bool bEnable = true) 
  { 
    m_bError = bEnable; 
  }

  inline bool errorModeEnabled() 
  { 
    return m_bError; 
  }

  inline void throwOdError(const OdError &pError)
  {
    enableErrorMode(true);
    throw pError;
  }

protected:
  HANDLE    m_hFile;
  OdString  m_sFileName;
  int       m_iFileShare;
  bool      m_bFileWritten;

  OdBaseFileBuf()
  {
    m_hFile = INVALID_HANDLE_VALUE;
    m_iFileShare = 0;
    m_bFileWritten = false;
    m_bError = false;
  }

public:
  /** Description:
    Opens the specified file with this StreamBuf object.
    Arguments:
    filename (I) Filename.
    shareMode (I) Share mode.
    accessMode (I) Access mode.
    creationDisposition (I) Creation disposition.
  */
  virtual void open(const OdString& filename,
    Oda::FileShareMode shareMode,
    Oda::FileAccessMode accessMode,
    Oda::FileCreationDisposition creationDisposition);
#ifndef DD_UNICODE  
  virtual void open(const OdCharW* filename,
    Oda::FileShareMode shareMode,
    Oda::FileAccessMode accessMode,
    Oda::FileCreationDisposition creationDisposition);
#endif

  /** Description:
    Closes the file opened by this StreamBuf object.
  */
  virtual void close();

  /** Description:
    Returns the name of the file associated with this StreamBuf object.
  */
  virtual OdString fileName() { return m_sFileName; }

  /** Description:
    Returns the share mode of this StreamBuf object.
  */
  virtual OdUInt32 getShareMode() { return OdUInt32(m_iFileShare); }
  
  /** Description:
    Returns the *length* in bytes of the file associated with
    this StreamBuf object.
  */
  virtual OdUInt64 length();
  
  /** Description:
    Moves the current position of the file pointer for
    this StreamBuf object to the specified location.

    Arguments:
    offset (I) Number of *bytes*.
    seekType (I) Seek type.
    
    Remarks:
    seekType must be one of the following:
    
    @table
    Name                    Value       Description.
    OdDb::kSeekFromStart    SEEK_SET    Start of file.
    OdDb::kSeekFromCurrent  SEEK_CUR    Current position of file pointer.
    OdDb::kSeekFromEnd      SEEK_END    End of file.
  */
  virtual OdUInt64 seek(OdInt64 offset, OdDb::FilerSeekType seekType);

  /** Description:
    Returns the current position of the file pointer for
    this StreamBuf object.
  */
  virtual OdUInt64 tell();

  /** Description:
    Returns true if and only if the current position of the file pointer for
    this StreamBuf object is at the end of file.
  */
  virtual bool isEof();

  /** Description:
    Returns the byte at the current position of the file pointer
    for this StreamBuf object, and increments the pointer.
  */
  virtual OdUInt8 getByte();

  /** Description:
    Returns the specified number of bytes, starting at the current position of the file pointer
    for this StreamBuf object, and increments the pointer by numBytes.
    Arguments:
    buffer (I) Character *buffer*.
    numBytes (I) Number of *bytes*.
  */
  virtual void getBytes(void* buffer, OdUInt32 numBytes);

  /** Description:
    Writes the byte to the current position of the file pointer
    for this StreamBuf object, and increments the pointer.
    Arguments:
    value (I) Byte to write. 
  */
  virtual void putByte(OdUInt8 value);

  /** Description:
    Writes the specified number of bytes, starting at the current position of the file pointer
    for this StreamBuf object, and increments the pointer by numBytes.
    Arguments:
    buffer (I) Character *buffer*.
    numBytes (I) Number of *bytes*.
  */
  virtual void putBytes(const void* buffer, OdUInt32 numBytes);

  /** Description:
    Sets the *length* of the file associated with
    this StreamBuf object to zero.
  */
  virtual void truncate();

   /** Description:
    Copies the specified bytes from this StreamBuf object to the specified StreamBuf object.
    Arguments:
    pDestination (I) Pointer to the StreamBuf object to receive the *data*.
    sourceStart (I) Starting position of the file pointer of this StreamBuf object.
    sourceEnd (I) Ending position of the file pointer of this StreamBuf object.
    Remarks:
    Bytes are copied from sourceStart to sourceEnd inclusive.
  */
 virtual void copyDataTo(OdStreamBuf* pDestination, OdUInt64 sourceStart, OdUInt64 sourceEnd);
};

class OdRdFileBuf;

/** Remarks:
  This template class is a specialization of the OdSmartPtr class for OdRdFileBuf object pointers.
*/
typedef OdSmartPtr<OdRdFileBuf> OdRdFileBufPtr;

/** Description:
  This class implements file input.
  Library:
  Source *code* provided.
  {group:ExServices_Classes}
*/
class OdRdFileBuf : public OdBaseFileBuf
{
  OdRdFileBuf(const OdRdFileBuf& source);
  OdRdFileBuf& operator = (const OdRdFileBuf& source);

protected:
  HANDLE      m_hFileMap;
  LPVOID      m_pFileMap;
  ULARGE_INTEGER  m_ulSize;
  ULARGE_INTEGER  m_ulPos;

  /** { Secret } */
  inline bool memBufferUsed() const { return m_pFileMap != 0; }

public:
  OdRdFileBuf(const OdString& filename) 
  { 
    m_hFileMap = INVALID_HANDLE_VALUE;
    m_pFileMap = NULL;
    m_ulSize.QuadPart = 0;
    m_ulPos.QuadPart = 0;

    open(filename); 
  }

  /** Description:
    Arguments:
    filename (I) Filename.
    shareMode (I) Share mode.
    accessMode (I) Access mode.
    creationDisposition (I) Creation disposition.
    Remarks:
    Opens filename if specified.
  */
  OdRdFileBuf(
    const OdString& filename, 
    Oda::FileShareMode shareMode, 
    Oda::FileAccessMode accessMode,
    Oda::FileCreationDisposition creationDisposition)
  { 
    m_hFileMap = INVALID_HANDLE_VALUE;
    m_pFileMap = NULL;
    m_ulSize.QuadPart = 0;
    m_ulPos.QuadPart = 0;

    open(filename, shareMode, accessMode, creationDisposition); 
  }

  OdRdFileBuf();

  ~OdRdFileBuf() { close(); }

  /** 
    Arguments:
    filename (I) Filename.
    shareMode (I) Share mode.
    accessMode (I) Access mode.
    creationDisposition (I) Creation disposition.
    Remarks:
    Opens filename if specified.
  */
  static OdRdFileBufPtr createObject()
  {
    return OdRdFileBufPtr(new OdRdFileBuf(), kOdRxObjAttach);
  }

  static OdRdFileBufPtr createObject(
    const OdString& filename, 
    Oda::FileShareMode shareMode = Oda::kShareDenyWrite,
    Oda::FileAccessMode accessMode = Oda::kFileRead,
    Oda::FileCreationDisposition creationDisposition = Oda::kOpenExisting)
  {
    return OdRdFileBufPtr(new OdRdFileBuf(filename, shareMode, accessMode, creationDisposition), kOdRxObjAttach);
  }

  virtual void open(const OdString& filename,
    Oda::FileShareMode shareMode = Oda::kShareDenyWrite,
    Oda::FileAccessMode accessMode = Oda::kFileRead,
    Oda::FileCreationDisposition creationDisposition = Oda::kOpenExisting);
#ifndef DD_UNICODE  
  virtual void open(const OdCharW* filename,
    Oda::FileShareMode shareMode = Oda::kShareDenyWrite,
    Oda::FileAccessMode accessMode = Oda::kFileRead,
    Oda::FileCreationDisposition creationDisposition = Oda::kOpenExisting);
#endif

  virtual void close();

  virtual OdUInt64 length();
  virtual OdUInt64 seek(OdInt64 offset, OdDb::FilerSeekType seekType);
  virtual OdUInt64 tell();
  virtual bool isEof();
  virtual OdUInt8 getByte();
  virtual void getBytes(void* buffer, OdUInt32 numBytes);
  virtual void putByte(OdUInt8 value);
  virtual void putBytes(const void* buffer, OdUInt32 numBytes);
  virtual void truncate();

  virtual void copyDataTo(OdStreamBuf* pDestination, OdUInt64 sourceStart, OdUInt64 sourceEnd);
};

#define WRITING_BUFFER_LENGTH 1024*8
class OdWrFileBuf;

/** Remarks:
  This template class is a specialization of the OdSmartPtr class for OdWrFileBuf object pointers.
*/
typedef OdSmartPtr<OdWrFileBuf> OdWrFileBufPtr;

/** Description:
  This class implements file output.
  Library:
  Source *code* provided.
  {group:ExServices_Classes}
*/
class OdWrFileBuf : public OdBaseFileBuf
{
  OdWrFileBuf(const OdWrFileBuf& source);
  OdWrFileBuf& operator = (const OdWrFileBuf& source);
  OdUInt8 m_pBuffer[WRITING_BUFFER_LENGTH];
  OdUInt32 m_nBufferedSize;
  inline void flush()
  {
    if ( m_nBufferedSize == 0 )
      return;
    OdBaseFileBuf::putBytes(m_pBuffer, m_nBufferedSize);
    m_nBufferedSize = 0;
  }

public:

  /** Description:
    Arguments:
    filename (I) Filename.
    shareMode (I) Share mode.
    accessMode (I) Access mode.
    creationDisposition (I) Creation disposition.
    Remarks:
    Opens filename if specified.
  */
  OdWrFileBuf(const OdString& filename):m_nBufferedSize(0) { open(filename); }
  OdWrFileBuf(
    const OdString& filename, 
    Oda::FileShareMode shareMode,
    Oda::FileAccessMode accessMode,
    Oda::FileCreationDisposition creationDisposition):m_nBufferedSize(0)
  { 
    open(filename, shareMode, accessMode, creationDisposition); 
  }

  OdWrFileBuf();

  ~OdWrFileBuf();

  /**
    Arguments:
    filename (I) Filename.
    shareMode (I) Share mode.
    accessMode (I) Access mode.
    creationDisposition (I) Creation disposition.
    Remarks:
    Opens filename if specified.
  */
  static OdWrFileBufPtr createObject()
  {
    return OdWrFileBufPtr(new OdWrFileBuf(), kOdRxObjAttach);
  }
  static OdWrFileBufPtr createObject(const OdString& filename,
    Oda::FileShareMode shareMode = Oda::kShareDenyNo,
    Oda::FileAccessMode accessMode = Oda::kFileWrite,
    Oda::FileCreationDisposition creationDisposition = Oda::kCreateAlways)
  {
    return OdWrFileBufPtr(new OdWrFileBuf(filename, shareMode, accessMode, creationDisposition), kOdRxObjAttach);
  }

  virtual void close();
  virtual OdUInt64 seek(OdInt64 offset, OdDb::FilerSeekType seekType);
  virtual void copyDataTo(OdStreamBuf* pDestination, OdUInt64 sourceStart, OdUInt64 sourceEnd);
  virtual OdUInt64 tell();
  virtual OdUInt64 length();

  virtual void putByte(OdUInt8 value);
  virtual void putBytes(const void* buffer, OdUInt32 numBytes);
  virtual OdUInt8 getByte();
  virtual void getBytes(void* buffer, OdUInt32 numBytes);

  virtual void open(const OdString& filename,
    Oda::FileShareMode shareMode = Oda::kShareDenyNo,
    Oda::FileAccessMode accessMode = Oda::kFileWrite,
    Oda::FileCreationDisposition creationDisposition = Oda::kCreateAlways);
#ifndef DD_UNICODE  
  virtual void open(const OdCharW* filename,
    Oda::FileShareMode shareMode = Oda::kShareDenyNo,
    Oda::FileAccessMode accessMode = Oda::kFileWrite,
    Oda::FileCreationDisposition creationDisposition = Oda::kCreateAlways);
#endif
};

#else // #ifdef WIN32

#include <stdio.h>
#include "DbSystemServices.h"
#include "OdString.h"
#include "RxObjectImpl.h"

#define ERR_VAL ((OdUInt32)-1)

class OdBaseFileBuf : public OdRxObjectImpl<OdStreamBuf>
{
  OdBaseFileBuf(const OdBaseFileBuf&);
  OdBaseFileBuf& operator = (const OdBaseFileBuf&);

public:
          OdBaseFileBuf();
  virtual ~OdBaseFileBuf()  {}

  virtual void      open(
    const OdString& filename, 
    Oda::FileShareMode shareMode   /* = Oda::kShareDenyNo*/, 
    Oda::FileAccessMode accessMode /* = Oda::kFileRead*/, 
    Oda::FileCreationDisposition creationDisposition /* = Oda::kOpenExisting*/) = 0;

  virtual void      close();
  virtual OdString  fileName() { return m_FileName; }

protected:
  FILE *              m_fp;
  OdString            m_FileName;
  OdUInt32            m_length;
  Oda::FileShareMode  m_shMode;

  void open(const OdString& path, const OdString& mode);
};

class OdWrFileBuf;
typedef OdSmartPtr<OdWrFileBuf> OdWrFileBufPtr;

class OdWrFileBuf : public OdBaseFileBuf
{
  OdWrFileBuf(const OdWrFileBuf& source);
  OdWrFileBuf& operator = (const OdWrFileBuf& source);

  OdUInt64  m_position;

public:
  OdWrFileBuf(const OdString& filename) { open(filename); }
  OdWrFileBuf(const OdString& filename, Oda::FileShareMode shareMode) { open(filename, shareMode); }
  OdWrFileBuf();
  ~OdWrFileBuf();
  
  static OdWrFileBufPtr createObject()
  {
    return OdWrFileBufPtr(new OdWrFileBuf(), kOdRxObjAttach);
  }
  static OdWrFileBufPtr createObject(const OdString& filename, Oda::FileShareMode shareMode = Oda::kShareDenyNo)
  {
    return OdWrFileBufPtr(new OdWrFileBuf(filename, shareMode), kOdRxObjAttach);
  }

  virtual void open(
    const OdString& filename, 
    Oda::FileShareMode shareMode = Oda::kShareDenyNo, 
    Oda::FileAccessMode accessMode = Oda::kFileWrite, 
    Oda::FileCreationDisposition creationDisposition = Oda::kCreateAlways);

  virtual void close();

  // OdStreamBuf methods
  virtual OdUInt64  length();
  virtual OdUInt64  seek(OdInt64 offset, OdDb::FilerSeekType seekType);
  virtual OdUInt64  tell();
  virtual bool      isEof();
  virtual void      putByte(OdUInt8 value);
  virtual void      putBytes(const void* buffer, OdUInt32 numBytes);
  virtual OdUInt32  getShareMode();
};


#define NUM_BUFFERS 8

class OdRdFileBuf;
typedef OdSmartPtr<OdRdFileBuf> OdRdFileBufPtr;

class OdRdFileBuf : public OdBaseFileBuf
{
  OdRdFileBuf(const OdRdFileBuf&);
  OdRdFileBuf& operator = (const OdRdFileBuf&);

public:
  OdRdFileBuf(const OdString& filename) : m_Counter(0L) { init(); open(filename); }
  OdRdFileBuf(const OdString& filename, Oda::FileShareMode shareMode) : m_Counter(0L)
  { 
    init();
    open(filename, shareMode); 
  }
  OdRdFileBuf();
  static OdRdFileBufPtr createObject()
  {
    return OdRdFileBufPtr(new OdRdFileBuf(), kOdRxObjAttach);
  }
  static OdRdFileBufPtr createObject(const OdString& filename, Oda::FileShareMode shareMode = Oda::kShareDenyNo)
  {
    return OdRdFileBufPtr(new OdRdFileBuf(filename, shareMode), kOdRxObjAttach);
  }
  ~OdRdFileBuf();

  virtual void open(
    const OdString& filename, 
    Oda::FileShareMode shareMode = Oda::kShareDenyNo, 
    Oda::FileAccessMode accessMode = Oda::kFileRead, 
    Oda::FileCreationDisposition creationDisposition = Oda::kOpenExisting);

  virtual void close();

  virtual OdUInt64  length();
  virtual OdUInt64  seek(OdInt64 offset, OdDb::FilerSeekType seekType);
  virtual OdUInt64  tell();
  virtual bool      isEof();
  virtual OdUInt8   getByte();
  virtual void      getBytes(void* buffer, OdUInt32 numBytes);
  virtual OdUInt32  getShareMode();

protected:
  struct blockstru
  {
    OdUInt8*  buf;        /* this buffer */
    OdUInt64  startaddr;  /* address from which it came in the file */
    int       validbytes; /* number of valid bytes it holds */
    OdInt32   counter;    /* least recently used counter */
  };

  OdUInt64  m_PhysFilePos; /* where the file pointer is */
  OdUInt64  m_BufPos;      /* position from which buf was filled */
  int       m_BytesLeft;   /* bytes left in buf */
  int       m_BufBytes;    /* valid bytes read into buffer */
  OdUInt8*  m_pNextChar;   /* pointer to next char in buffer */
  OdUInt8*  m_pCurBuf;     /* pointer to the buffer currently being used */
  int       m_UsingBlock;  /* which block is currently in use */
  struct blockstru         /* the data being held */
            m_DataBlock[NUM_BUFFERS];

  static const int m_BufSize; /* size of each read buffer */
  static const int  m_PosMask; /* mask to allow position check */
  OdInt32    m_Counter;

  bool filbuf();
  void init();
};

#endif // #ifdef WIN32

#endif // ODFILEBUF_DEFINED


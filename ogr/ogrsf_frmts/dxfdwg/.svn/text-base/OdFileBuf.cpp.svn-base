
#if (defined(WIN32) || defined(WIN64)) && !defined(_WIN32_WCE)

#define ODA_NON_TRACING   // Comment it to have trace

#pragma warning(disable: 4290)

#if defined(_MSC_VER)
#include <windows.h>
#endif

#include "OdaCommon.h"
#include "OdFileBuf.h"
#include "OdString.h"


void OdBaseFileBuf::open(const OdString& wszFileName, Oda::FileShareMode nShareMode, Oda::FileAccessMode nDesiredAccess, Oda::FileCreationDisposition nCreationDisposition)
{
	close();
	ODA_TRACE(DD_T("open('%ls', 0x%x, 0x%x, %d)"), wszFileName.c_str(), nDesiredAccess, nShareMode, nCreationDisposition);

	// Check file name passed
  if( wszFileName.isEmpty() ) throw OdError(eNoFileName);
	
	DWORD dwAccess = 0, dwAttr = 0;
	if( nDesiredAccess & Oda::kFileRead )
	{
		dwAccess |= GENERIC_READ;
		dwAttr |= FILE_FLAG_RANDOM_ACCESS;
	}
	if( nDesiredAccess & Oda::kFileWrite )
	{
		dwAccess |= GENERIC_WRITE;
		dwAttr |= FILE_ATTRIBUTE_NORMAL;
	}

	DWORD dwShare = 0;
	if( nShareMode != Oda::kShareDenyRead && nShareMode != Oda::kShareDenyReadWrite )
	{
		dwShare |= FILE_SHARE_READ;
	}
	if( nShareMode != Oda::kShareDenyWrite && nShareMode != Oda::kShareDenyReadWrite )
	{
		dwShare |= FILE_SHARE_WRITE;
	}

	DWORD dwDisposition = 0;
	switch( nCreationDisposition )
	{
		case Oda::kCreateNew:
			dwDisposition = CREATE_NEW;
			break;
		case Oda::kCreateAlways:
			dwDisposition = CREATE_ALWAYS;
			break;
		case Oda::kOpenExisting:
			dwDisposition = OPEN_EXISTING;
			break;
		case Oda::kOpenAlways:
			dwDisposition = OPEN_ALWAYS;
			break;
		case Oda::kTruncateExisting:
			dwDisposition = TRUNCATE_EXISTING;
			break;
	}

  OdString sFileName = wszFileName;
  m_hFile =	::CreateFileW(wszFileName, dwAccess, dwShare, NULL, dwDisposition, dwAttr, NULL);

	if( m_hFile == INVALID_HANDLE_VALUE )
	{
    ODA_TRACE1("open() can't open file, GetLastError = %x", ::GetLastError());
		throw OdError_FileException(eCantOpenFile, sFileName);
	}
	else
	{
		m_sFileName = sFileName;
		m_iFileShare = nShareMode;
	}

	//ATLTRACE("OdBaseFileBuf::open(%hs)\n", m_sFileName.c_str());
}

void OdBaseFileBuf::close()
{
	if( m_hFile != INVALID_HANDLE_VALUE )
	{
		ODA_TRACE(DD_T("close()"));
		// Flush buffer if writing was done
		if( m_bFileWritten ) ::FlushFileBuffers(m_hFile);
		::CloseHandle(m_hFile);
		m_hFile = INVALID_HANDLE_VALUE;
	}
	m_sFileName.empty();
	m_iFileShare = 0;
	m_bFileWritten = false;
  m_bError = false;
}

OdUInt64 OdBaseFileBuf::length()
{
	ODA_TRACE(DD_T("length()"));
	return ::GetFileSize(m_hFile, NULL);
}

#if !defined(INVALID_SET_FILE_POINTER) // for _MSC_VER <= 1200
#define INVALID_SET_FILE_POINTER ((DWORD)-1)
#endif

OdUInt64 OdBaseFileBuf::seek(OdInt64 offset, OdDb::FilerSeekType whence)
{
	ODA_TRACE(DD_T("seek(%d, %d)"), offset, whence);

	DWORD dwMethod = 0;
	switch( whence )
	{
		case OdDb::kSeekFromStart:
			dwMethod = FILE_BEGIN;
			break;
		case OdDb::kSeekFromCurrent:
			dwMethod = FILE_CURRENT;
			break;
		case OdDb::kSeekFromEnd:
			dwMethod = FILE_END;
			break;
	}

  LARGE_INTEGER li;
  li.QuadPart = offset;
  li.LowPart = ::SetFilePointer(m_hFile, li.LowPart, &li.HighPart, dwMethod);
  if (li.LowPart == INVALID_SET_FILE_POINTER && ::GetLastError() != NO_ERROR)
  {
    li.QuadPart = -1;
    throw OdError_FileException(eFileInternalErr, m_sFileName);
  }
  return li.QuadPart;
}

OdUInt64 OdBaseFileBuf::tell()
{
	ODA_TRACE(DD_T("tell()"));
	return ::SetFilePointer(m_hFile, 0, NULL, FILE_CURRENT);
}

bool OdBaseFileBuf::isEof()
{
	ODA_TRACE(DD_T("isEof()"));
	return (::GetFileSize(m_hFile, NULL) == ::SetFilePointer(m_hFile, 0, NULL, FILE_CURRENT));
}

OdUInt8 OdBaseFileBuf::getByte()
{
	ODA_TRACE(DD_T("getByte()"));

	DWORD dwBytes = 0;
	OdUInt8 b = 0;
	if( !::ReadFile(m_hFile, &b, 1, &dwBytes, NULL) )
	{
		ODA_TRACE(DD_T("getByte() can't read byte due to read error"));
		throw OdError_FileException(eFileInternalErr, m_sFileName);
	}
	if( dwBytes == 0 )
	{
		ODA_TRACE(DD_T("getByte() can't read byte due to end-of-file"));
		throw OdError_FileException(eEndOfFile, m_sFileName);
	}
	
	return b;
}

void OdBaseFileBuf::getBytes(void* buffer, OdUInt32 nLen)
{
	ODA_TRACE(DD_T("getBytes(%p, %u)"), buffer, nLen);

	DWORD dwBytes = 0;
	if( !::ReadFile(m_hFile, buffer, nLen, &dwBytes, NULL) )
	{
		ODA_TRACE(DD_T("getBytes() can't read byte due to read error"));
		throw OdError_FileException(eFileInternalErr, m_sFileName);
	}
	if( dwBytes < nLen )
	{
		ODA_TRACE(DD_T("getBytes() read only %u bytes due to end-of-file"), dwBytes);
		throw OdError_FileException(eEndOfFile, m_sFileName);
	}
}

void OdBaseFileBuf::putByte(OdUInt8 val)
{
	ODA_TRACE(DD_T("putByte(0x%x)"), val);

	DWORD dwBytes = 0;
	BYTE b = val;
	if( !::WriteFile(m_hFile, &b, 1, &dwBytes, NULL) || dwBytes != 1 )
	{
		ODA_TRACE(DD_T("putByte() can't write byte"));
		throwOdError(OdError_FileException(eFileWriteError, m_sFileName));
	}
	m_bFileWritten = true;
}

void OdBaseFileBuf::putBytes(const void* buffer, OdUInt32 nLen)
{
	ODA_TRACE(DD_T("putBytes(%p, %u)"), buffer, nLen);

	DWORD dwBytes = 0;
	if( !::WriteFile(m_hFile, buffer, nLen, &dwBytes, NULL) || dwBytes != nLen )
	{
		ODA_TRACE(DD_T("putBytes() can't write bytes"));
		throwOdError(OdError_FileException(eFileWriteError, m_sFileName));
	}
	m_bFileWritten = true;
}

void OdBaseFileBuf::truncate()
{
	ODA_TRACE(DD_T("truncate()"));
	if( !::SetEndOfFile(m_hFile) )
	{
		ODA_TRACE(DD_T("truncate() failed"));
		throw OdError_FileException(eFileInternalErr, m_sFileName);
	}
}

// Current method behavior description (so required behavior isn't documented in DWGDirect yet):
//  - Move source stream to nSrcStart position
//  - Read (nSrcEnd-nSrcStart) bytes from source stream to temporary buffer
//  - Write buffer to target stream
//  - Set source stream to nSrcEnd position
void OdBaseFileBuf::copyDataTo(OdStreamBuf* pDest, OdUInt64 nSrcStart, OdUInt64 nSrcEnd)
{
	ODA_TRACE(DD_T("copyDataTo(%p, %u, %u)"), pDest, nSrcStart, nSrcEnd);

  if(nSrcStart==0 && nSrcEnd==0)
  {
    nSrcStart = tell();
    nSrcEnd = length(); 
  }

  // Do nothing if incorrect positions passed
	if( nSrcEnd <= nSrcStart ) return;

	// Remember current position to restore on error
	OdUInt64 nPos = tell();
	if( nPos != nSrcStart )
	{
		// Move to start position
		if( seek(nSrcStart, OdDb::kSeekFromStart) != nSrcStart )
		{
			ODA_TRACE(DD_T("copyDataTo() can't move to start position"));
			throw OdError_FileException(eEndOfFile, m_sFileName);
		}
	}

	// Allocate temporary buffer
	OdUInt32 nLen = OdUInt32(nSrcEnd - nSrcStart);
	
	HLOCAL hBuf = ::LocalAlloc(LHND, nLen);
	if( hBuf == NULL )
	{
		ODA_TRACE(DD_T("copyDataTo() can't allocate temporary buffer"));
		seek(nPos, OdDb::kSeekFromStart);
		throw OdError_FileException(eOutOfMemory, m_sFileName);
	}

	void* pBuf = ::LocalLock(hBuf);
	if( pBuf == NULL )
	{
		ODA_TRACE(DD_T("copyDataTo() can't access to temporary buffer"));
		::LocalFree(hBuf);
		seek(nPos, OdDb::kSeekFromStart);
		throw OdError_FileException(eOutOfMemory, m_sFileName);
	}

	// Read to buffer (current position will be set to nSrcEnd after)
	getBytes(pBuf, nLen);
	//seek(nPos, OdDb::kSeekFromStart);

	// Write to destination
	pDest->putBytes(pBuf, nLen);

	::LocalUnlock(hBuf);
	::LocalFree(hBuf);
}


OdWrFileBuf::OdWrFileBuf() : m_nBufferedSize(0)
{
}

OdWrFileBuf::~OdWrFileBuf() 
{ 
  close(); 
}

#ifndef DD_UNICODE
void OdWrFileBuf::open(const OdChar* szFileName, Oda::FileShareMode nShareMode, Oda::FileAccessMode nDesiredAccess, Oda::FileCreationDisposition nCreationDisposition)
{
  OdWString wsStr = OdStringToWideChar(szFileName);
  if (wsStr.isEmpty())
    throw OdError(eNoFileName);

  OdBaseFileBuf::open(wsStr.c_str(), nShareMode, nDesiredAccess, nCreationDisposition);
}
#endif

#ifndef DD_UNICODE
void OdWrFileBuf::open(const OdCharW* wszFileName, Oda::FileShareMode nShareMode, Oda::FileAccessMode nDesiredAccess, Oda::FileCreationDisposition nCreationDisposition)
#else
void OdWrFileBuf::open(const OdString& wszFileName, Oda::FileShareMode nShareMode, Oda::FileAccessMode nDesiredAccess, Oda::FileCreationDisposition nCreationDisposition)
#endif
{
  if (nDesiredAccess == Oda::kFileRead)
    throw OdError_FileException(eCantOpenFile, m_sFileName);

  OdBaseFileBuf::open(wszFileName, nShareMode, nDesiredAccess, nCreationDisposition);
}

#ifndef DD_UNICODE
void OdRdFileBuf::open(const OdChar* szFileName, Oda::FileShareMode nShareMode, Oda::FileAccessMode nDesiredAccess, Oda::FileCreationDisposition nCreationDisposition)
{
  OdWString wsStr = OdStringToWideChar(szFileName);
  if (wsStr.isEmpty())
    throw OdError(eNoFileName);

  open(wsStr.c_str(), nShareMode, nDesiredAccess, nCreationDisposition);
}
void OdRdFileBuf::open(const OdCharW* wszFileName, Oda::FileShareMode nShareMode, Oda::FileAccessMode nDesiredAccess, Oda::FileCreationDisposition nCreationDisposition)
#else
void OdRdFileBuf::open(const OdString& wszFileName, Oda::FileShareMode nShareMode, Oda::FileAccessMode nDesiredAccess, Oda::FileCreationDisposition nCreationDisposition)
#endif
{
  if (nDesiredAccess & Oda::kFileWrite)
    throw OdError_FileException(eCantOpenFile, m_sFileName);

  OdBaseFileBuf::open(wszFileName, nShareMode, nDesiredAccess, nCreationDisposition);

	// Test open mode and file attributes to use fast mapping buffer instead of plain one
	if( nCreationDisposition == Oda::kOpenExisting || nCreationDisposition == Oda::kOpenAlways )
	{
		// Reject compressed and other special files
		// TODO Perhaps reject remote files also
		DWORD dwAttr =
#ifdef DD_UNICODE
			::GetFileAttributesW(wszFileName);
#else
			::GetFileAttributesA(OdWideCharToString((LPCWSTR)wszFileName));
#endif //DD_UNICODE
		if( dwAttr != INVALID_FILE_SIZE/*INVALID_FILE_ATTRIBUTES*/ &&
			!(dwAttr & (FILE_ATTRIBUTE_DIRECTORY|FILE_ATTRIBUTE_ENCRYPTED|
			FILE_ATTRIBUTE_SPARSE_FILE|FILE_ATTRIBUTE_REPARSE_POINT|
			FILE_ATTRIBUTE_COMPRESSED|FILE_ATTRIBUTE_OFFLINE)) )
		{
			try
			{
        // Remember actual file size
      	m_ulSize.LowPart = ::GetFileSize(m_hFile, &m_ulSize.HighPart);

      	DWORD dwProtect = PAGE_READONLY, dwAccess = FILE_MAP_READ;
      	if( nDesiredAccess & Oda::kFileWrite )
      	{
      		dwProtect = PAGE_READWRITE;
      		dwAccess = FILE_MAP_WRITE;
      	}

      	// Map file into memory
      	m_hFileMap = ::CreateFileMapping(m_hFile, NULL, dwProtect, 0, 0, NULL);
      	if( m_hFileMap == INVALID_HANDLE_VALUE ) throw OdError_FileException(eCantOpenFile, m_sFileName);

      	m_pFileMap = ::MapViewOfFile(m_hFileMap, dwAccess, 0, 0, 0);
      	if( !m_pFileMap ) throw OdError_FileException(eCantOpenFile, m_sFileName);
			}
			catch( ... )
			{
				// Fall down to plain buffer use
				// disable cache
			}
		}
	}
}

OdRdFileBuf::OdRdFileBuf()
{
	m_hFileMap = INVALID_HANDLE_VALUE;
	m_pFileMap = NULL;
	m_ulSize.QuadPart = 0;
	m_ulPos.QuadPart = 0;
}

void OdRdFileBuf::close()
{
	if( m_pFileMap )
	{
		::UnmapViewOfFile(m_pFileMap);
		m_pFileMap = NULL;
	}
	if( m_hFileMap != INVALID_HANDLE_VALUE )
	{
		::CloseHandle(m_hFileMap);
		m_hFileMap = INVALID_HANDLE_VALUE;
	}
	m_ulSize.QuadPart = 0;
	m_ulPos.QuadPart = 0;

	OdBaseFileBuf::close();
}

OdUInt64 OdRdFileBuf::length()
{
  return memBufferUsed() ? OdUInt64(m_ulSize.QuadPart) : OdBaseFileBuf::length();
}

OdUInt64 OdRdFileBuf::seek(OdInt64 offset, OdDb::FilerSeekType whence)
{
  if (!memBufferUsed())
    return OdBaseFileBuf::seek(offset, whence);

	switch( whence )
	{
		case OdDb::kSeekFromStart:
			if( offset < 0 || offset > (OdInt64)m_ulSize.QuadPart ) throw OdError_FileException(eFileInternalErr, m_sFileName);
			m_ulPos.QuadPart = offset;
			break;
		case OdDb::kSeekFromCurrent:
			if( (m_ulPos.QuadPart + offset) < 0 || (m_ulPos.QuadPart + offset) > m_ulSize.QuadPart ) throw OdError_FileException(eFileInternalErr, m_sFileName);
			m_ulPos.QuadPart += offset;
			break;
		case OdDb::kSeekFromEnd:
			if( offset > 0 || -offset > (OdInt64)m_ulSize.QuadPart ) throw OdError_FileException(eFileInternalErr, m_sFileName);
			m_ulPos.QuadPart = m_ulSize.QuadPart + offset;
			break;
	}

	return m_ulPos.QuadPart;
}

OdUInt64 OdRdFileBuf::tell()
{
  if (!memBufferUsed())
    return OdBaseFileBuf::tell();

	return OdUInt64(m_ulPos.QuadPart);
}

bool OdRdFileBuf::isEof()
{
  if (!memBufferUsed())
    return OdBaseFileBuf::isEof();

  return (m_ulPos.QuadPart == m_ulSize.QuadPart);
}

OdUInt8 OdRdFileBuf::getByte()
{
  if (!memBufferUsed())
    return OdBaseFileBuf::getByte();

  if( m_ulPos.QuadPart >= m_ulSize.QuadPart ) throw OdError_FileException(eEndOfFile, m_sFileName);

	return ((OdUInt8*)m_pFileMap)[m_ulPos.QuadPart++];
}

void OdRdFileBuf::getBytes(void* buffer, OdUInt32 nLen)
{
  if (!memBufferUsed())
  {
    OdBaseFileBuf::getBytes(buffer, nLen);
    return;
  }

  if( (m_ulPos.QuadPart + nLen) > m_ulSize.QuadPart ) throw OdError_FileException(eEndOfFile, m_sFileName);

	::CopyMemory(buffer, ((OdUInt8*)m_pFileMap) + m_ulPos.QuadPart, nLen);

	m_ulPos.QuadPart += nLen;
}

void OdRdFileBuf::putByte(OdUInt8 val)
{
  if (!memBufferUsed())
  {
    OdBaseFileBuf::putByte(val);
    return;
  }

  if( m_ulPos.QuadPart > m_ulSize.QuadPart ) throw OdError_FileException(eFileWriteError, m_sFileName);

	((OdUInt8*)m_pFileMap)[m_ulPos.QuadPart++] = val;

	// TODO? m_bFileWritten = true;
}

void OdRdFileBuf::putBytes(const void* buffer, OdUInt32 nLen)
{
  if (!memBufferUsed())
  {
    OdBaseFileBuf::putBytes(buffer, nLen);
    return;
  }

  if( (m_ulPos.QuadPart + nLen) > m_ulSize.QuadPart ) throw OdError_FileException(eFileWriteError, m_sFileName);

	::CopyMemory(((OdUInt8*)m_pFileMap) + m_ulPos.QuadPart, buffer, nLen);

	m_ulPos.QuadPart += nLen;

	// TODO? m_bFileWritten = true;
}

void OdRdFileBuf::truncate()
{
  if (!memBufferUsed())
  {
    OdBaseFileBuf::truncate();
    return;
  }

  throw OdError_FileException(eFileWriteError, m_sFileName);
}

// Current method behavior description (so required behavior isn't documented in DWGDirect yet):
//  - Move source stream to nSrcStart position
//  - Read (nSrcEnd-nSrcStart) bytes from source stream to temporary buffer
//  - Write buffer to target stream
//  - Set source stream to nSrcEnd position
void OdRdFileBuf::copyDataTo(OdStreamBuf* pDest, OdUInt64 nSrcStart, OdUInt64 nSrcEnd)
{
  if (!memBufferUsed())
  {
    OdBaseFileBuf::copyDataTo(pDest, nSrcStart, nSrcEnd);
    return;
  }

  if( !pDest ) throw OdError_FileException(eNullObjectPointer, m_sFileName);

  if(nSrcStart==0 && nSrcEnd==0)
  {
    nSrcStart = tell();
    nSrcEnd = length(); 
  }

  // Do nothing if incorrect positions passed
	if( nSrcEnd <= nSrcStart ) return;

	if( nSrcEnd > m_ulSize.QuadPart ) throw OdError_FileException(eEndOfFile, m_sFileName);

	pDest->putBytes(((OdUInt8*)m_pFileMap) + nSrcStart, OdUInt32(nSrcEnd - nSrcStart));
}

OdUInt64 OdWrFileBuf::seek(OdInt64 offset, OdDb::FilerSeekType whence)
{
  flush();
  return OdBaseFileBuf::seek(offset, whence);
}

 void OdWrFileBuf::putByte(OdUInt8 val)
{
  if ( m_nBufferedSize == WRITING_BUFFER_LENGTH )
  {
    flush();
  }
  m_pBuffer[m_nBufferedSize++] = val;
}

void OdWrFileBuf::putBytes(const void* buffer, OdUInt32 nLen)
{
  if (nLen > WRITING_BUFFER_LENGTH)
  {
    flush();
    OdBaseFileBuf::putBytes(buffer, nLen);
    return;
  }
  if ( m_nBufferedSize + nLen > WRITING_BUFFER_LENGTH )
  {
    flush();
  }
  memcpy(m_pBuffer + m_nBufferedSize, buffer, nLen);
  m_nBufferedSize += nLen;
}

OdUInt64 OdWrFileBuf::tell()
{
  return OdBaseFileBuf::tell() + m_nBufferedSize;
}

void OdWrFileBuf::close()
{
  if (!errorModeEnabled()) 
  // to prevent second exception in putBytes. cr 3861
  {
    flush();
  }
  OdBaseFileBuf::close();
}

OdUInt8 OdWrFileBuf::getByte()
{
  flush();
  return OdBaseFileBuf::getByte();
}
void OdWrFileBuf::getBytes(void* buffer, OdUInt32 nLen)
{
  flush();
  OdBaseFileBuf::getBytes(buffer,nLen);
}
void OdWrFileBuf::copyDataTo(OdStreamBuf* pDest, OdUInt64 nSrcStart, OdUInt64 nSrcEnd)
{
  flush();
  OdBaseFileBuf::copyDataTo(pDest, nSrcStart,nSrcEnd);
}

OdUInt64 OdWrFileBuf::length()
{
  OdUInt64 l = OdBaseFileBuf::length();
  OdUInt64 pos = OdBaseFileBuf::tell();
  if ( pos + m_nBufferedSize > l )
  {
    return pos + m_nBufferedSize;
  }
  else  
  {
    return l;
  }
}

#else // #ifdef WIN32

#include <stdlib.h>
#include "OdaCommon.h"
#include "OdFileBuf.h"


//////////////////////////////////////////////////////////////////////
// Construction/Destruction
//////////////////////////////////////////////////////////////////////

OdBaseFileBuf::OdBaseFileBuf()
    : m_fp(0)
	  , m_FileName("")
    , m_length(ERR_VAL)
    , m_shMode(Oda::kShareDenyNo)
{}

void OdBaseFileBuf::close()
{
  m_length = ERR_VAL;
  m_FileName = "";
  if (m_fp)
  {
    if (fclose(m_fp) != 0)
		{
      ODA_FAIL(); // eFileCloseError;
		}
    m_fp = 0;
  }
}

#ifdef _WIN32_WCE
#define _IOFBF          0x0000
#endif

void OdBaseFileBuf::open(const OdString& fileName, const OdString& access)
{
  m_fp = fopen((const char*)fileName, (const char*)access);
  if (m_fp)
  {
    setvbuf(m_fp, 0, _IOFBF, 8192);
    m_FileName = fileName;
  }
  else
    throw OdError_CantOpenFile(fileName);
}


////////////////////////////////////////////////////////////////////
OdWrFileBuf::OdWrFileBuf()
    : m_position(ERR_VAL)
{
}

OdWrFileBuf::~OdWrFileBuf()
{
  close();
}


void OdWrFileBuf::open(
  const OdString& path, 
  Oda::FileShareMode /*shMode*/, 
  Oda::FileAccessMode /*nDesiredAccess*/, 
  Oda::FileCreationDisposition /*nCreationDisposition*/)
{
  OdString  sMode = DD_T("wb");

  //if (shMode == Oda::kShareDenyReadWrite)

  m_position = 0;
  m_length = 0;
  OdBaseFileBuf::open(path, sMode);
  return;
}

void OdWrFileBuf::close()
{
  m_position = ERR_VAL;
  OdBaseFileBuf::close();
}

OdUInt64 OdWrFileBuf::length()
{
  return m_length;
}

OdUInt64 OdWrFileBuf::seek(OdInt64 offset, OdDb::FilerSeekType whence)
{
  switch (whence) {
  case OdDb::kSeekFromStart:
    m_position = offset;
    break;
  case OdDb::kSeekFromCurrent:
    m_position += offset;
    break;
  case OdDb::kSeekFromEnd:
    m_position = m_length - offset;
    break;
  }
  if (fseek(m_fp, m_position, SEEK_SET) != 0)
    m_position = ERR_VAL;  // Error
  return m_position;
}

OdUInt64 OdWrFileBuf::tell()
{
  return m_position;
}

bool OdWrFileBuf::isEof()
{
  return (m_position >= m_length);
}

OdUInt32 OdWrFileBuf::getShareMode()
{
  return (OdUInt32)m_shMode;
}

void OdWrFileBuf::putByte(OdUInt8 val)
{
	if(::fputc(val,m_fp)==EOF)
	{
    throw OdError_FileWriteError(m_FileName);
	}
  if (++m_position > m_length)
    m_length = m_position;
}

void OdWrFileBuf::putBytes(const void* buff, OdUInt32 nByteLen)
{
	if(::fwrite(buff, 1, nByteLen, m_fp) < nByteLen)
	{
    throw OdError_FileWriteError(m_FileName);
	}
  m_position += nByteLen;
  if (m_position > m_length)
    m_length = m_position;
}


////////////////////////////////////////////////////////////////////////////////
const int OdRdFileBuf::m_BufSize = 8192;
const int OdRdFileBuf::m_PosMask(~(8192-1));


OdRdFileBuf::OdRdFileBuf()
    : m_Counter(0L)
{
  init();
}

void OdRdFileBuf::init()
{
   for (int i = 0; i < NUM_BUFFERS; i++)
  {
    m_DataBlock[i].buf = NULL;
    m_DataBlock[i].counter = -1L;
    m_DataBlock[i].validbytes=0;
    m_DataBlock[i].startaddr = ERR_VAL;
  }
}


OdRdFileBuf::~OdRdFileBuf()
{
  close();
}

void OdRdFileBuf::close()
{
  // indicate buffers no longer in use
  for (int i = 0; i < NUM_BUFFERS; i++)
  {
    if (m_DataBlock[i].buf)
      ::odrxFree(m_DataBlock[i].buf);
    m_DataBlock[i].buf = NULL;
    m_DataBlock[i].counter = -1L;
    m_DataBlock[i].validbytes=0;
    m_DataBlock[i].startaddr = ERR_VAL;
  }
  OdBaseFileBuf::close();
}

void OdRdFileBuf::open(
  const OdString& fname, 
  Oda::FileShareMode shMode,
  Oda::FileAccessMode /*nDesiredAccess*/, 
  Oda::FileCreationDisposition /*nCreationDisposition*/)
{
  OdString caMode;

  if (shMode == Oda::kShareDenyWrite || shMode == Oda::kShareDenyReadWrite)
    caMode = DD_T("r+b");
  else 
    caMode = DD_T("rb");

  OdBaseFileBuf::open(fname, caMode);

  // Get file length
  OdUInt64 curLoc = ftell(m_fp);
	fseek(m_fp, 0L, 2);
	m_length = ftell(m_fp);
  fseek(m_fp, curLoc, 0);

  int i;

  m_BufBytes= 0;
  m_BytesLeft= 0;
  m_BufPos=0L;  // to start reads from 0L
  m_pCurBuf=NULL;
  m_pNextChar = m_pCurBuf;
  m_UsingBlock = -1;
  m_PhysFilePos=0L;

  for (i = 0; i < NUM_BUFFERS; i++)
  {
    m_DataBlock[i].buf = (OdUInt8*)::odrxAlloc(m_BufSize);
    m_DataBlock[i].validbytes=0;
    m_DataBlock[i].counter = -1L;
    m_DataBlock[i].startaddr = ERR_VAL;
  }
  seek(0L, OdDb::kSeekFromStart);  // initial seek, gets a buffer & stuff
  return;
}

bool OdRdFileBuf::filbuf( )
{
  int i, minindex;
  OdInt32 minlru;
  struct blockstru *minptr;

  m_UsingBlock = -1;
  /* see if we are holding it already */
  for (i = 0; i < NUM_BUFFERS; i++)
  {
    if (m_DataBlock[i].startaddr==m_BufPos)
      break;
  }

  if (i < NUM_BUFFERS)   // we are already holding this part of the file
  {
    m_pCurBuf=m_DataBlock[i].buf;
    m_BufPos=m_DataBlock[i].startaddr;
    m_BytesLeft=m_BufBytes=m_DataBlock[i].validbytes;
    m_pNextChar=m_pCurBuf;
    m_DataBlock[i].counter=m_Counter++;
    m_UsingBlock=i;
    return true;
  }

  /* not holding it, so look for a buffer to read into */
  /* first see if any are not yet loaded */
  minptr=NULL;
  minindex=0;

  for (i = 0; i < NUM_BUFFERS; i++)
  {
    if (m_DataBlock[i].startaddr==ERR_VAL)
    {
      minindex=i;
      minptr=&m_DataBlock[i];
      break;
    }
  }

  /* if all were used, then look for the least recently used one */
  if (minptr==NULL)
  {
    minlru=0x7FFFFFFF;
    minptr=NULL;
    minindex=0;

    for (i = 0; i < NUM_BUFFERS; i++) {
      if (m_DataBlock[i].counter<0L)
        m_DataBlock[i].counter=0L;
      if (m_DataBlock[i].counter<minlru) {
        minlru=m_DataBlock[i].counter;
        minptr=&m_DataBlock[i];
        minindex=i;
      }
    }
  }

  if (minptr==NULL)
  {
    ODA_FAIL();
    return false;  /* couldn't find one */
  }
  /* if we are not already physically at the read location, move there */
  /* then read into the buffer */
  if (m_PhysFilePos!=m_BufPos /*|| readerror*/)
  {
    fseek(m_fp, m_BufPos, SEEK_SET);
  }
  m_BufBytes = m_BytesLeft =
    (short) fread(minptr->buf, 1, m_BufSize, m_fp);
  m_PhysFilePos=m_BufPos+m_BufBytes;
  if (m_BufBytes > 0)
  {
    minptr->validbytes = m_BufBytes;
    minptr->startaddr = m_BufPos;
    minptr->counter=m_Counter++;
    m_pCurBuf=minptr->buf;
    m_pNextChar = m_pCurBuf;
    m_UsingBlock=minindex;
    return true;
  }
  return false;
}

OdUInt32 OdRdFileBuf::getShareMode()
{
  return (OdUInt32)m_shMode;
}

OdUInt64 OdRdFileBuf::length()
{
  return m_length;
}


OdUInt64 OdRdFileBuf::seek(OdInt64 offset, OdDb::FilerSeekType whence)
{
  int bytestoadvance;

  switch (whence)
  {
  case OdDb::kSeekFromStart:
    break;
  case OdDb::kSeekFromCurrent:
    offset += (m_BufPos+(m_pNextChar - m_pCurBuf));
    break;
  case OdDb::kSeekFromEnd:
    offset = m_length - offset;
    break;
  }

  ODA_ASSERT(offset >= 0);
  // from here on assume whence is 0
  // if it's not in the area we're holding, seek to it
  if (OdUInt64(offset) < m_BufPos || OdUInt64(offset) >= m_BufPos + m_BufBytes)
  {
    m_BufPos=offset & m_PosMask;
    if (!filbuf( )) // locates it if we're already holding in another block
    {
      m_pNextChar = NULL;
      m_pCurBuf = NULL;
      m_BytesLeft = 0;
      throw OdError(eEndOfFile);
      //return ERR_VAL;
    }
  }
  m_pNextChar = (m_pCurBuf + (bytestoadvance=(OdUInt16)(offset - m_BufPos)));
  m_BytesLeft = m_BufBytes - bytestoadvance;
  return(offset);
}

OdUInt64 OdRdFileBuf::tell()
{
  return (m_BufPos + (m_pNextChar - m_pCurBuf));
}


bool OdRdFileBuf::isEof()
{
  if (m_BytesLeft > 0)
    return false;
  if (m_length == 0)
    return true;
  m_BufPos += m_BufBytes;
  return !filbuf();
}


OdUInt8 OdRdFileBuf::getByte()
{
  m_DataBlock[m_UsingBlock].counter=m_Counter++;
  if (m_BytesLeft<=0) {
    m_BufPos+=m_BufBytes;
    if (!filbuf())
    {
      throw OdError(eEndOfFile);
    }
  }
  m_BytesLeft--;
  return (*m_pNextChar++);
}

void OdRdFileBuf::getBytes(void* buffer, OdUInt32 nLen)
{
  OdInt32 bytesleft;
  OdUInt16 bytestoread;
  unsigned char *buf=(unsigned char *)buffer;

  if (nLen > 0)
  {
    m_DataBlock[m_UsingBlock].counter=m_Counter++;
    bytesleft = nLen;

    while (bytesleft > 0L && !isEof( )) {
      if ((OdInt32)m_BytesLeft<bytesleft) bytestoread=(OdUInt16)m_BytesLeft;
      else bytestoread=(OdUInt16)bytesleft;

      memcpy(buf,m_pNextChar,bytestoread);
      m_BytesLeft -= bytestoread;
      m_pNextChar += bytestoread;
      buf += bytestoread;
      bytesleft -= bytestoread;
    }
    if (bytesleft > 0L)
      throw OdError(eEndOfFile);
  }
}
 
#endif // #ifdef WIN32


ZZIPlib Library 
==================
한컴오피스 한/워드에서 압축 라이브러리 용도로 ZZIPlib Library(http://sourceforge.net/zzip-index.html) 사용함에 따라 MPL 1.1 License 의 소스 공개 준수의무 이행을 위해 공개한 소스입니다.

본 자료는 hancom.com 사이트의 고객지원 > 개발자센터 > 개발자료실(게시물 제목 : '12. [소스] zzip 소스코드 다운로드',  등록일 2017-11-01) )에서 공개했던 자료를 옮겨 등록한 자료입니다. 

변경사항
==================

* 기존 개발소스와의 충돌 회피를 위해 zip.c 파일을 zzip.c 으로 파일명 변경 

* 다음 파일 리스트에 대해 #include를 통한 header파일 포함시 대상 header파일 위치를 로컬로 변경 


>> 변경 파일 목록

__debug.h
__dirent.h
__fnmatch.h
__hints.h
__mmap.h
conf.h
dir.c
err.c
fetch.c
fetch.h
file.c
file.h
format.h
fseeko.c
fseeko.h
info.c
lib.h
memdisk.c
memdisk.h
mmapped.c
mmapped.h
plugin.c
plugin.h
stat.c
stdint.h
types.h
write.c
write.h
zzip.h
zzip32.h



>> 변경 사항  

 #include "__debug.h"
 #include "__dirent.h"
 #include "__fnmatch.h"
 #include "__hints.h"
 #include "__mmap.h"
 #include "conf.h"
 #include "conf.h"
 #include "fetch.h"
 #include "file.h"
 #include "format.h"
 #include "fseeko.h"
 #include "info.h"
 #include "lib.h"
 #include "lib.h"
 #include "memdisk.h"
 #include "mmapped.h"
 #include "plugin.h"
 #include "stdint.h"
 #include "types.h"
 #include "write.h"
 #include "zzip.h"
 #include "zzip32.h"


* 다음 파일들에 대해 각각 설명된바와 같이 운영체제별 빌드를 위한 directive처리가 추가되었습니다.

>>  __dirent.h

다음과 같이 telldir ,  seekdir 에 대한 define 은 OSX에서만 define되도록 변경 

#ifdef OS_MAC
#define _zzip_telldir   telldir
#define _zzip_seekdir   seekdir
#endif


그리고, 다음과 같이 윈도우즈용 빌드에 대해 보안강화 를 위한 strcpy 함수의 strcpy_s 으로의 변경이 반영되었습니다.

#ifdef _WIN32
    strcpy_s (nd->dd_name, strlen(szPath) + 1, szPath);
#else
	strcpy(nd->dd_name, szPath);
#endif



>>  __fnmatch.h

정상빌드를 위해 다음과 같이 string.h 헤더를 추가하였습니다.

#include <string.h>


>>  _config.h

OSX에서의 정상빌드를 위해 다음과 같이 ZZIP_HAVE_BYTESWAP_H 와 ZZIP_HAVE_STRNDUP 에 대한 정의를 차단하였습니다.

#ifndef OS_MAC
#ifndef ZZIP_HAVE_BYTESWAP_H 
#define ZZIP_HAVE_BYTESWAP_H  1 
#endif
#endif

#ifndef OS_MAC
#ifndef ZZIP_HAVE_STRNDUP
#define ZZIP_HAVE_STRNDUP  1
#endif
#endif


>> conf.h

clang으로 빌드되는 경우 빌드오류를 회피하기위해 다음과 같이 조건에 따라 define되도록 하였습니다.

#ifdef __clang__
#define _zzip_inline
#else
#define _zzip_inline inline
#endif /* __clang__ */


>>  dir.c

윈도우즈 빌드의 경우에 대해, API 보안이슈 회피를 위해 다음과 같이 처리하였습니다.

#ifdef _WIN32
	strcpy_s(filename, PATH_MAX, dir->realname);
	strcat_s(filename, PATH_MAX, "/");
	strcat_s(filename, PATH_MAX, dirent->d_name);
#else
	strcpy(filename, dir->realname);
	strcat(filename, "/");
	strcat(filename, dirent->d_name);
#endif

그리고, OSX에 대한 빌드가 아닌경우 zzip_telldir, zzip_seekdir 함수에 대한 구현부분 정의가 
다음과 같은 형식으로 skip되도록 directive 분기처리 하였습니다.

#ifdef OS_MAC
......
zzip_off_t
zzip_telldir(ZZIP_DIR * dir)
......
void
zzip_seekdir(ZZIP_DIR * dir, zzip_off_t offset)
......
#endif



>>  err.c

윈도우즈 빌드의 경우에 대해,  API보안이슈에 대한 처리(stderr_s 대체사용) 및 관련 코드가 추가되었습니다.

zzip_char_t *
zzip_strerror(int errcode)
{
	static char error[256];
	memset(error, 0, 256);

    if (errcode < ZZIP_ERROR && errcode > ZZIP_ERROR - 32)
    {
        struct errlistentry *err = errlist;

        for (; err->mesg; err++)
        {
            if (err->code == errcode)
                return err->mesg;
        }
        errcode = EINVAL;
    }

#ifdef _WIN32
	if (errcode < 0)
	{
		if (errcode == -1) {
			strerror_s(error, 256, errcode);
			return error;
		} else {
			return zError(errcode);
		}
	}
	strerror_s(error, 256, errcode);
	return error;
#else
    if (errcode < 0)
    {
		if (errcode == -1) {
			strerror_r(errcode, error, 256);
			return error;
		} else {
			return zError(errcode);
		}
    }
	strerror_r(errcode, error, 256);
	return error;
#endif
}


zzip_char_t *
zzip_strerror_of(ZZIP_DIR * dir)
{
	static char error[256];
	memset(error, 0, 256);

	if (!dir) {
#ifdef _WIN32
		strerror_s(error, 256, errno);
		return error;
#else
		strerror_r(errno, error, 256);
		return error;
#endif

	}
    return zzip_strerror(dir->errcode);
}


>>  fseeko.c

PAGESIZE 로 정의된 directive를 PAGE_SIZE 로 변경하여 빌드오류를 회피하도록 하였습니다.



>> mmapped.c

윈도우즈 빌드의 경우에 대해, API보안이슈 회피를 위한 처리 (strncpy_s 대체사용) 가 추가되었습니다.

#ifdef _WIN32
	strncpy_s(r, maxlen + 1, p, maxlen);
#else
	strncpy(r, p, maxlen);
#endif



>>  plugin.c

윈도우즈 빌드의 경우에 대해, zzip_plugin_io default_io 에 사용되는 open 메소드를 다른 메소드 (hwopen) 로 재정의하였습니다.

#ifdef OS_WIN
#include <Windows.h>
#include <share.h>
int hwopen(const char * filename, int flag)
{
	int res;
	WCHAR wfilename[_MAX_PATH];
	if (MultiByteToWideChar(CP_UTF8, 0, filename, -1, wfilename, _MAX_PATH) == 0)
		return -1;

	_wsopen_s(&res, wfilename, flag, _SH_DENYNO, _S_IREAD | _S_IWRITE);

	return res;
}
#endif //OS_WIN

static const struct zzip_plugin_io default_io = {
#ifdef OS_WIN
	&hwopen,
    &_close,
#else
    &open,
    &close,
#endif
    &_zzip_read,
    &_zzip_lseek,
    &zzip_filesize,
    1, 1,
    &_zzip_write
};



>>  zzip.h

다음과 같이 OSX빌드에서만 취급되도록 directive 분기 처리 되었습니다.

#ifdef OS_MAC
#define zzip_telldir zzip_telldir64
#define zzip_seekdir zzip_seekdir64
#endif


#ifdef OS_MAC
_zzip_export
zzip_off_t  	zzip_telldir(ZZIP_DIR * dir);
_zzip_export
void	 	zzip_seekdir(ZZIP_DIR * dir, zzip_off_t offset);
#endif



>> zzip32.h

다음과 같이 OSX빌드에서만 취급되도록 directive 분기 처리 되었습니다.

#ifdef OS_MAC
_zzip_export
long zzip_telldir32(ZZIP_DIR * dir);
_zzip_export
void zzip_seekdir32(ZZIP_DIR * dir, long offset);
#endif


라이선스
==================
이 프로젝트는 MPL-1.1 라이선스를 따릅니다. 자세한 고지 내용은 아래 파일들을 참고하세요.
* [zzip_notice.txt](https://github.com/hancom-io/oss-download/blob/main/zzip/zzip_notice.txt)
* [zziplib_Modify.txt](https://github.com/hancom-io/oss-download/blob/main/zzip/zziplib_Modify.txt)


첨부파일 : [zzip.zip](https://github.com/hancom-io/oss-download/blob/main/zzip/zzip.zip)

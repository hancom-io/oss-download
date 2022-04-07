libcairo 
==================
한컴오피스에서 svg 이미지를 문서 화면에 표현하기 위한 용도로 libcairo(https://cairographics.org/) 사용함에 따라 MPL 1.1 License 의 소스 공개 준수의무 이행을 위해 공개한 소스입니다.

본 자료는 hancom.com 사이트의 고객지원 > 개발자센터 > 개발자료실(게시물 제목 : '11.[소스] libcairo 소스코드 다운로드',  등록일 2017-11-01) )에서 공개했던 자료를 옮겨 등록한 자료입니다. 

변경사항
==================
libpng 라이브러리 업데이트로 인해 변경된 libpng 구조체에 맞추기 위한 코드 수정

>> 변경 파일 목록
cairo-png.c - 2017.07.07

>> 변경 사항
※추가※
#include <pngpriv.h> 추가

※변경※
<원본>
png_simple_error_callback (png_structp png_save_ptr,
                   png_const_charp error_msg)
{
    _cairo_error (CAIRO_STATUS_NO_MEMORY);
    longjmp (png_save_ptr->jmpbuf, CAIRO_STATUS_NO_MEMORY);
}

<수정>
png_simple_error_callback (png_structp png_save_ptr,
                   png_const_charp error_msg)
{
    _cairo_error (CAIRO_STATUS_NO_MEMORY);
longjmp (png_save_ptr->jmp_buf_local, CAIRO_STATUS_NO_MEMORY);
}

(png_save_ptr->jmpbuf, 에서 png_save_ptr->jmp_buf_local로 변경)


라이선스
==================
이 프로젝트는 MPL-1.1 라이선스를 따릅니다. 자세한 고지 내용은 libcairo 1.4.10.0_notice(github 링크연결), libcairo 1.4.10.0_COPYING-MPL-1.1(github링크연결), libcairo 1.4.10.0_COPYING-LGPL-2.1(github 링크연결), libcairo_Modify.txt(github링크연결) 파일을 참고하세요.


첨부파일 :  libcairo 1.4.10.0.zip // ' libcairo 1.4.10.0.zip' 클릭시 Github에 올린  libcairo 1.4.10.0.zip 위치한 GitHub 페이지로 이동
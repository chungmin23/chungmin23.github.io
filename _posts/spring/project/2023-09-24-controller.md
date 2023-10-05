---
layout: single
title: "project"
categories: project
tag: project
toc: true
---

## controller

~~~
package com.study.admin.controller;

import com.study.admin.service.FileService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.core.io.FileSystemResource;
import org.springframework.http.HttpStatus;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.*;

import org.springframework.core.io.Resource;
import java.io.File;

@Controller
@RequestMapping("/admin/file")
public class FileController {

    @Autowired
    private FileService fileService;

    @Value("${spring.servlet.multipart.location}")
    private String uploadPath;

    @GetMapping(value = "/download", produces = MediaType.APPLICATION_OCTET_STREAM_VALUE)
    @ResponseBody
    public ResponseEntity<Resource> downloadFile(String filename){
        String filePath = uploadPath + filename;
        Resource resource = (Resource) new FileSystemResource(filePath);

        return new ResponseEntity<>(resource, HttpStatus.OK);
    }


    @PostMapping(value = "/deleteFile")
    public ResponseEntity<String> deleteFile(String filename, int id){

        fileService.removeFile(id);

        String filePath = uploadPath + filename;
        File file = new File(filePath);
        if(file.delete()==true)
            return new ResponseEntity<>(HttpStatus.OK);
        else
            return new ResponseEntity<>(HttpStatus.NOT_FOUND);
    }
}

package com.study.admin.controller;

import com.study.admin.dto.*;
import com.study.admin.enums.BoardType;
import com.study.admin.service.BoardService;
import com.study.admin.service.CategoryService;
//import com.study.admin.service.FreeService;
import com.study.admin.service.CommentService;
import com.study.admin.service.FileService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.core.io.FileSystemResource;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;

import javax.annotation.Resource;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpSession;
import java.io.IOException;
import java.util.List;

@Controller
@RequestMapping("/admin/free")
public class FreeController {

//    @Autowired
//    private FreeService freeService;

    @Autowired
    private CategoryService categoryService;

    @Autowired
    private BoardService boardService;

    @Autowired
    private CommentService commentService;


    @Autowired
    private FileService fileService;



    @GetMapping("/list")
    public String geFreeList(Model model, HttpServletRequest request, SearchDto searchDto){
        if(searchDto.getStartDate() != null || searchDto.getEndDate() != null){
            searchDto.setStartDate(searchDto.getStartDate()+" 00:00:00");
            searchDto.setEndDate(searchDto.getEndDate()+" 23:59:59");
        }

        BoardDto boardDto = new BoardDto();
        boardDto.setBoardType("FR");
        searchDto.setBoardType("FR");



        int totalCnt = boardService.getSearchTotalCnt(searchDto);

        List<BoardDto> boardList = boardService.getNoticeList(searchDto);

        PageHandler pageHandler = new PageHandler(totalCnt, searchDto);


        // 로그인
        HttpSession session = request.getSession();
        AdminDto adminDto = (AdminDto) session.getAttribute("loginUser");


        //카테고리 조회
        List<CategoryDto> categoryList = categoryService.getCategoryList("FR");
        String type = "free";


        model.addAttribute("searchDto", searchDto);
        model.addAttribute("categoryList",categoryList);
        model.addAttribute("admin",adminDto);
        model.addAttribute("board",boardList);
        model.addAttribute("ph",pageHandler);
        model.addAttribute("type",type);

        return "board";
    }

      /**
     * 수정 페이지 조회
     * @param seq
     * @param model
     * @return
     */
    @GetMapping("/view/{seq}")
    public String getWriteForm(@PathVariable("seq") int seq, Model model, HttpServletRequest request){

        BoardDto board = boardService.getBoardForm(seq);
        List<CategoryDto> categoryList = categoryService.getCategoryList("FR");
        boardService.addViewCnt(seq);

        String type = "free";

        HttpSession session = request.getSession();
        AdminDto adminDto = (AdminDto) session.getAttribute("loginUser");

        List<FileDto> files =   fileService.fileList(seq);

        List<CommentDto> commentList =  commentService.getCommentList(seq);

        model.addAttribute("files",files);
        model.addAttribute("commentList",commentList);
        model.addAttribute("admin",adminDto);
        model.addAttribute("categoryList",categoryList);
        model.addAttribute("board",board);
        model.addAttribute("formType","update");
        model.addAttribute("type",type);
        return "boardDetail";
    }



    /**
     * 공지항사 수정
     * @param seq
     * @param boardDto
     * @return
     */
    @PostMapping(value = "/modify/{seq}")
    public String modifyFreeForm(@PathVariable("seq") int seq, BoardDto boardDto, MultipartFile file) throws IOException {
        boardDto.setBoardId(seq);
        boardDto.setTitle(boardDto.getTitle());
        boardDto.setContent(boardDto.getContent());
        boardDto.setCategoryId(boardDto.getCategoryId());

        fileService.fileUpload(boardDto.getFiles(),boardDto.getBoardId());



        return "redirect:/admin/free/list";
    }




}

~~~

## dto

~~~
package com.study.admin.dto;

import lombok.Builder;
import lombok.Data;

@Builder
@Data
public class FileDto {

    private Integer fileId;
    private Integer boardId;

    private String originalFileName;

    private String storedFileName;

    private String filePath;

    private String fileSize;

    private String thumbnailName;

}
package com.study.admin.dto;

import lombok.Data;
import org.springframework.web.multipart.MultipartFile;

import java.util.List;

@Data
public class BoardDto {

    private Integer boardId; // 게시판 아이디

    private Integer rownum; // 게시판 번호

    private String title; // 제목

    private String content; // 내용

    private int viewCnt; // 조회수

    private String boardType; // 게시판 타입

    private String createdDate; // 생성일

    private String name; // 이름
    
    private char alarm = 'N'; // 알림 여부
    
    private int adminId; // 관리자 아이디

    private String categoryName; // 카테고리 이름

    private int categoryId; // 카테고리 아이디

    private String answer; // 답변

    private MultipartFile[] files; //


}


~~~

## mapper

~~~
package com.study.admin.mapper;

import com.study.admin.dto.CategoryDto;
import com.study.admin.dto.FileDto;
import org.apache.ibatis.annotations.Mapper;

import java.util.List;

@Mapper
public interface FileMapper {

    void fileUpload(FileDto fileDto);


    List<FileDto> selectFileList(int seq);

    void deleteFile(int seq);
}

package com.study.admin.mapper;

import com.study.admin.dto.BoardDto;
import org.apache.ibatis.annotations.Mapper;
import org.springframework.beans.factory.annotation.Autowired;

@Mapper
public interface FreeMapper {

    BoardDto selectFreeList(int seq);


}

package com.study.admin.mapper;

import com.study.admin.dto.BoardDto;
import com.study.admin.dto.SearchDto;
import org.apache.ibatis.annotations.Mapper;

import java.util.List;

@Mapper
public interface BoardMapper {

    /**
     * 공지사항 조회
     * @return
     */
    List<BoardDto> selectNoticeList(SearchDto searchDto);

    /**
     * 공지사항 업데이트
     * @param id
     */
    void updateViewCnt(int id);

    /**
     * 공지사항 폼 조회
     * @param seq
     * @return
     */
    BoardDto selectBoardForm(int seq);


    int selectTotalCnt(SearchDto searchDto);


    /**
     * 공지사항 삭제
     * @param seq
     */
    void deleteBoard(int seq);

}

~~~


## service


~~~
package com.study.admin.service;

import com.study.admin.dto.CommentDto;
import com.study.admin.dto.FileDto;
import com.study.admin.mapper.FileMapper;
import net.coobird.thumbnailator.Thumbnails;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Service;
import org.springframework.web.multipart.MultipartFile;

import javax.imageio.ImageIO;
import java.awt.image.BufferedImage;
import java.io.File;
import java.io.IOException;
import java.util.List;
import java.util.UUID;

@Service
public class FileService {
    @Autowired
    private FileMapper fileMapper;

    @Value("${spring.servlet.multipart.location}")
    private String uploadPath;


    public void fileUpload(MultipartFile[] files, int boardId) throws IOException {

        if(files.length == 0) {
            return ;
        }

        for(MultipartFile file : files){
            String originalName = file.getOriginalFilename();


            String storeFilename = UUID.randomUUID() + "." + extractExt(originalName);
            String filePath = uploadPath + storeFilename;

            File upload = new File(filePath);
            file.transferTo(upload);

            String thumbnailName = "s_" + storeFilename;

            FileDto fileDto = FileDto.builder()
                    .storedFileName(storeFilename)
                    .originalFileName(originalName)
                    .fileSize(String.valueOf(file.getSize()))
                    .thumbnailName(thumbnailName)
                    .boardId(boardId)
                    .filePath(filePath)
                    .build();


            fileMapper.fileUpload(fileDto);


            File thumbnailFile = new File(uploadPath, thumbnailName);

            BufferedImage bo_img = ImageIO.read(upload);
            double ratio = 3;
            int width = (int) (bo_img.getWidth() / ratio);
            int height = (int) (bo_img.getHeight() / ratio);

            Thumbnails.of(upload)
                    .size(width, height)
                    .toFile(thumbnailFile);
        }


    }

    private String extractExt(String originalFilename) {
        int pos = originalFilename.lastIndexOf(".");
        return originalFilename.substring(pos + 1);
    }


    public List<FileDto> fileList(int seq){
       return   fileMapper.selectFileList(seq);
    }

    public void removeFile(int seq) { fileMapper.deleteFile(seq);}


}




package com.study.admin.service;

import com.study.admin.dto.BoardDto;
import com.study.admin.mapper.FreeMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class FreeService {


    @Autowired
    private FreeMapper freeMapper;


    public BoardDto getFreeList(int seq) {return freeMapper.selectFreeList(seq);}
}



~~~

## mapper

~~~
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.study.admin.mapper.FileMapper">
    <insert id="fileUpload" parameterType="com.study.admin.dto.FileDto">
        INSERT INTO file
        (board_id,original_file_name,stored_file_name,file_path, file_size,thumbnail_name, created_date )
        VALUES
        (#{boardId},#{originalFileName},#{storedFileName},#{filePath},#{fileSize},#{thumbnailName},now())

    </insert>


    <select id="selectFileList" parameterType="int" resultType="com.study.admin.dto.FileDto">
        SELECT *
        FROM file
        WHERE board_id = #{seq}
    </select>


    <delete id="deleteFile" parameterType="int">
        DELETE FROM file
        WHERE file_id = #{seq}
    </delete>
</mapper>

<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.study.admin.mapper.FreeMapper">


</mapper>



~~~

## template


~~~
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<html lang="kr">
<head>
    <script src="/js/common.js"></script>

    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.2/css/bootstrap.min.css">
    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.2/css/bootstrap-theme.min.css">
    <script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.2/js/bootstrap.min.js"></script>
    <script src="https://code.jquery.com/jquery-latest.min.js"></script>
    <link rel="stylesheet" href="/css/common.css">
    <script src="/js/board.js"></script>
    <meta charset="UTF-8">
    <title>Title</title>
</head>


<body>
 <main id="resultContainer">
     <header>
         <div><th:block th:text="${admin.getName()}"></th:block> 님 안녕하세요</div>
         <a  class="btn btn-link"  th:onclick="|location.href='@{/admin/logout}'|" >로그아웃</a>
     </header>
     <nav class="side-bar">
         <ul>
             <li><a th:href="@{/admin/notice/list}">공지사항 관리</a></li>
             <li><a th:href="@{/admin/free/list}">자유 게시판 관리</a></li>
             <li><a th:href="@{/admin/gallery/list}">갤러리 게시판 관리</a></li>
             <li><a th:href="@{/admin/qna/list}">문의 게시판 관리</a></li>
         </ul>
     </nav>
     <div class="container">
         <th th:switch="${type}">
             <h2 th:case="notice">공지사항</h2>
             <h2 th:case="free">자유 게시판</h2>
             <h2 th:case="gallery">갤러리 게시판</h2>
             <h2 th:case="qna">문의 게시판</h2>
         </th>

          <form id="forms" method="get" >
         <input type="hidden" name="page" id="pageInput">
         <input type="hidden" name="boardType" id="boardType">
         <div class="menu-bar">

                 <h4 style="display:inline-block;">등록일시</h4>
                 <div style="display:inline-block;">
                  <input type="date" name="startDate" id="startDate"/> ~
                  <input type="date" name="endDate" id="endDate"/>
                 </div>
                     <th:block th:if="${type != 'qna'}">
                         <select  class="searchField" name="searchField">
                            <option value="all">전체 분류</option>
                            <th:block th:each="category: ${categoryList}">
                                <option  th:selected="${searchDto.searchField}==${category.categoryId}" th:value="${category.categoryId}" th:text="${category.name}"></option>
                            </th:block>
                        </select>
                     </th:block>
                 <input type="text" class="inputs" placeholder="제목 or 내용" name="searchWord"/>
                 <button type="button" class="btn btn-default"  th:attr="data-type=${type}" th:onclick="sortSearch(this.getAttribute('data-type'))">검색</button>

         </div>
         <div class="select-bar">
             <div class="list-count">
                 <select name="limitCount" onchange="sortChange()"  >
                     <option th:selected="${searchDto.limitCount}==10" value=10>10</option>
                     <option th:selected="${searchDto.limitCount}==20" value=20>20</option>
                     <option th:selected="${searchDto.limitCount}==30" value=30>30</option>
                     <option th:selected="${searchDto.limitCount}==40" value=40>40</option>
                     <option th:selected="${searchDto.limitCount}==50" value=50>50</option>
                 </select>
                 <div></div>
             </div>
             <div class="list-sort">
                 <select name="searchSort" onchange="sortChange()">
                     <option th:selected="${searchDto.searchSort}=='createdDate'" value="createdDate">등록일시</option>
                     <option th:selected="${searchDto.searchSort}=='category'" value="category">분류</option>
                     <option th:selected="${searchDto.searchSort}=='title'" value="title">제목</option>
                     <option th:selected="${searchDto.searchSort}=='viewCnt'" value="viewCnt">조회수</option>
                 </select>
                 <select name="sort" onchange="sortChange()">
                     <option th:selected="${searchDto.sort}=='desc'" value=desc>내림차순</option>
                     <option th:selected="${searchDto.sort}=='asc'" value=asc>오름차순</option>
                 </select>
             </div>
         </div>
         </form>
         <th:block th:if="${type != 'qna'}">
             <div class="btn-insert">
                 <a class="btn btn-default" th:href="@{/admin/{type}/view(type=${type})}" >글 등록</a>
             </div>
         </th:block>
         <div th:replace="table :: table"></div>
         <div th:replace="page :: page"></div>

     </div>
 </main>
</body>
</html>


---

<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<html lang="en">
<head>
  <meta charset="UTF-8">
  <link rel="stylesheet" href="/css/common.css">
  <title>Title</title>
  <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.2/css/bootstrap.min.css">
  <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.2/css/bootstrap-theme.min.css">
  <script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.2/js/bootstrap.min.js"></script>
  <script src="https://code.jquery.com/jquery-latest.min.js"></script>
  <link rel="stylesheet" href="/css/boardDetail.css">
  <script src="/js/boardDetail.js"></script>
</head>
<body>
 <main>
   <th th:switch="${type}">
     <h3 th:case="notice">공지사항 관리</h3>
     <h3 th:case="free">자유 게시판 관리</h3>
     <h3 th:case="gallery">갤러리 게시판 관리</h3>
     <h3 th:case="qna">문의 게시판 관리</h3>
   </th>

   <form method="post" id="noticeForm" th:action="${formType == 'update' ? '/admin/'+type+'/modify/' + board.boardId : '/admin/'+type+'/write'}"   enctype="multipart/form-data">
     <table class="table">
       <input type="hidden" id="adminId" name="adminId" th:value="${admin.adminId}"/>
       <input type="hidden" id="name"  name="name" th:value="${admin.name}"/>
       <th:block th:if="${formType == 'update'}">
        <input type="hidden" id="boardId"  name="boardId" th:value="${board.boardId}"/>
       </th:block>

       <tbody>
       <th:block th:if="${type != 'qna'}">
         <tr>
           <td>분류</td>
           <td>
             <label>
               <select  class="searchField" name="categoryId"   >
                 <th:block th:each="category: ${categoryList}">
                   <option th:selected="${category.categoryId}"   th:value="${category.categoryId}" th:text="${category.name}"  ></option>
                 </th:block>

               </select>
             </label>
           </td>
         </tr>
       </th:block>
         <tr>
           <td>제목</td>
           <td><input type="text" class="text-max" name="title" th:value="${formType == 'update'} ? ${board.title} : '' "></td>
         </tr>
         <tr>
           <td th:text="${type == 'qna'} ? '질문' : '내용'">내용</td>
           <td><textarea class="text-max" name="content" th:value="${formType == 'update'} ? ${board.content} : '' "    th:text="${formType == 'update'} ? ${board.content} : '' "></textarea></td>
         </tr>
         <th:block th:if="${type == 'qna'}">
           <tr>
             <td>답변</td>
             <td><textarea class="text-max" name="answer"  th:value="${board.answer}" th:text="${board.answer}"></textarea></td>
           </tr>
         </th:block>
         <th:block th:if="${type == 'notice'}">
         <tr>
           <td>알림글</td>
           <td><input type="checkbox" name="alarm" onchange="YnCheck(this);" th:checked="${formType == 'update'} ? (${board.alarm} == 'Y') : false"   ></td>
         </tr>
         </th:block>
         <th:block th:if="${type == 'free'}">
           <td>첨부파일</td>
           <td>
               <div class="file_list">
                <th:block th:if="${files != null}">
                  <th:block th:each="file : ${files}">
                    <div class="file_input">
                      <label>
                        <a th:href="@{/admin/file/download(filename=${file.getStoredFileName()})}" type="file"  name="files" onchange="return filechk(this);" th:text="${file.getOriginalFileName()}" ></a>

                      </label>
                      <button type="button" th:attr="data-type=${file.getStoredFileName()}, id = ${file.getFileId()}"  onclick="deleteFile(this.getAttribute('id'),this, this.getAttribute('data-type'));" class="btns del_btn"><span>x</span></button>
                    </div>
                  </th:block>
                </th:block>


                 <div class="file_input">
<!--                   <label>-->
<!--                     <input type="file"  name="files" id="files" onchange="selectFile();" />-->

<!--                   </label>-->
<!--                   <button type="button" onclick="removeFile(this);" class="btns del_btn"><span>x</span></button>-->
                 </div>
               </div>

                 <button type="button" onclick="addFile();" class="btns fn_add_btn"><span>추가</span></button>

           </td>
         </th:block>


       </tbody>
     </table>
   </form>
     <th:block th:if="${type == 'free'}">

       <div class="container">
         <div class="row">
           <div style="display:flex; border: 1px solid #dddddd; padding: 10px; ">
             <textarea type="text" style="height:60px;" id="replyContents" class="form-control" name="commentText"></textarea>
             <button id="replySubmit" onclick="commentWrite()" style="width:60px; height:60px; margin-left:10px">등록하기</button>
           </div>
         </div>
         <div class="row">
           <table style="text-align:center; border:1px solid #dddddd; width: 100%;">
             <tbody>
             <tr th:each="comment : ${commentList}">
               <td>
                 <div style="display:flex;">
                   <span th:text="${comment.createdDate}" style="margin-right:5px;"></span>
                   <span th:text="${comment.commentName}" ></span>
                 </div>
                 <div style="text-align:left;">
                   <span th:text="${comment.content}"></span>
                 </div>
               </td>
               <td style="vertical-align:middle;"><button id="replyRemove"  style="widht:40px; height:40px;"th:attr="data-type=${comment.commentId}"  th:onclick="commentRemove(this.getAttribute('data-type'))">삭제</button></td>
             </tr>
             </tbody>


           </table>
         </div>
       </div>

     </th:block>

   <div class="buttons">
     <ul>
       <li th:if="${formType == 'insert'}"><button type="button" class="btn btn-primary"   id="insertNotice"  >등록</button></li>
       <li th:unless="${formType == 'insert'}"><button type="submit" th:text="${type == 'qna'} ? '답변완료' : '수정'" class="btn btn-primary" id="updateNotice" >수정</button></li>
       <li><button type="button" class="btn btn-default" th:onclick="|location.href='@{/admin/{type}/list(type=${type})}'|" >목록</button></li>
       <li th:if="${formType == 'update'}" ><button type="button" class="btn btn-danger" th:onclick="|location.href='@{/admin/{type}/delete/{id}(id=${board.boardId},type=${type})}'|">삭제</button></li>
     </ul>
   </div>
 </main>
</body>
</html>


~~~


## js

~~~
-- board

window.onload =function (){
    let now = new Date();
    let oneMonthAge = new Date(now.setMonth(now.getMonth() - 1));
    document.getElementById('startDate').value = oneMonthAge.toISOString().substring(0,10);
    document.getElementById('endDate').value = new Date().toISOString().substring(0,10);

}
document.addEventListener('keydown', function(event) {
    if (event.keyCode === 13) {
        event.preventDefault();
    };
}, true);

function sortChange() {
    let pageInput = document.getElementById('pageInput');

    // 입력 필드에 데이터 설정
    pageInput.value = 1;

    $('#forms').submit(); //form Submit
}


function sortSearch(boardType) {
    let pageInput = document.getElementById('pageInput');


    // 입력 필드에 데이터 설정
    pageInput.value = 1;
    let url = "/admin/"+boardType+"/list";


    $("#forms").attr("action", url);
    $('#forms').submit(); //form Submit
}

function pageForms(page) {

    let subForm = document.getElementById('forms');
    let pageInput = document.getElementById('pageInput');

    // 입력 필드에 데이터 설정
    pageInput.value = page;

    subForm.submit(); //form Submit
}



-- boarddetail

$(document).ready(function() {
    $("#updateNotice").click(function() {
        $("#noticeForm").submit();
    });
    $("#insertNotice").click(function() {
        $("#noticeForm").submit();
    });


});

function YnCheck(obj){
    let checked = obj.checked;
    if(checked){
        obj.value = 'Y';
    }else{
        obj.value = 'N';
    }
};

function filechk(obj){
    let fileSize = obj.files[0].size;
    let maxSize = 2 * 1024 * 1024; //100mb
    let ext = $(obj).val().split(".").pop().toLowerCase();

    //파일명에 특수문자 검사 (정규 표현식 사용)
    let fileName = $(obj).val().split("\\").pop(); // 파일 경로에서 파일명 추출
    let specialChars = /[*|\":<>[\]{}`\\()';@&$]/; // 특수문자를 나타내는 정규 표현식

    //파일용량 체크
    if(fileSize > maxSize){
        alert("파일 크기가 너무 큽니다. 최대 크기는 2MB입니다.");
        $(obj).val('');
        return false;
    }
    //파일 확장자 체크
    if($.inArray(ext, ["jpg", "jpeg", "png", "gif", "zip"]) == -1) {
        alert("올바른 파일 확장자가 아닙니다. 허용된 확장자는 jpg, jpeg, png, gif, zip 입니다.");
        $(obj).val('');
        return false;
    }

    if (obj.files && obj.files[0]) {
        let reader = new FileReader();
        reader.onload = function(e) {
            document.getElementById('preview').src = e.target.result;
        };
        reader.readAsDataURL(obj.files[0]);
    } else {
        document.getElementById('preview').src = "";
    }
}



function deleteFile(id,file, filename){



    removeFile(file);

    $.ajax({
        url:"/admin/file/deleteFile",
        data : {filename : filename, id : id},
        dataType : "text",
        type : 'post',
        success: function (result){
            alert(result);
        }
    })
}

// 파일 추가
function addFile() {
    const fileDiv = document.createElement('div');
    fileDiv.innerHTML =`
            <div class="file_input">
                    <img id="preview">
                   <label>
                     <input type="file"  name="files" id="files" onchange="filechk(this);" />

                   </label>
                   <button type="button" onclick="removeFile(this);" class="btns del_btn"><span>x</span></button>
                 </div>
        `;
    document.querySelector('.file_list').appendChild(fileDiv);
}


// 파일 삭제
function removeFile(element) {
    const fileAddBtn = element.nextElementSibling;
    if (fileAddBtn) {
        const inputs = element.previousElementSibling.querySelectorAll('input');
        inputs.forEach(input => input.value = '')
        return false;
    }
    element.parentElement.remove();



}





const commentWrite =() => {
    let replyContents = $("#replyContents").val();
    let boardId = $('#boardId').val();
    let userId = $('#adminId').val();
    let name =  $('#name').val();
    $.ajax({
        type : "POST",
        url : "/admin/comment/write",
        data : {
            "replyContents" : replyContents,
            "boardId" : boardId,
            "userId" : userId,
            "name" : name
        },
        contentType: "application/x-www-form-urlencoded",  // 폼 데이터 전송 설정
        dataType: "text",
        success : function(result) {
            if (result === "s") {
                alert("등록 성공");
                location.reload();
            }

        },
        error : function() {
            alert("등록 실패")

        }
    })
}

const commentRemove =(id) => {

    $.ajax({
        type : "DELETE",
        url : "/admin/comment/delete/"+id,
        dataType: "text",
        success : function(result) {
            if (result === "s") {
                alert("삭제 성공");
                location.reload();
            }

        },
        error : function() {
            alert("삭제 실패")

        }
    })
}



~~~


## css

~~~
--common 
*{
 padding : 0;
 margin: 0;
 list-style-type: none;
}

header {
 position: fixed;
 top:0;
 width:100%;
 height:64px;
 border-bottom: 1px solid #ebebff;
 background: #fff;
 z-index:30;
 float : right;
 display:flex;
 align-items:end;
 justify-content: right;
 padding:0 10px 20px 0;

}

header div {
 padding-right: 10px;
}

nav{
 padding: 64px 30px 0 30px;
 position: absolute;
 left:0;
 top:0;
 height:100%;
 z-index:20;

}

.container{
  padding:64px 0 0 150px;
  width: calc(100% - 150px);
}

.side-bar {
 text-align: center;
 font-weight: bold;
 border: 1px solid #ebebff
}

.side-bar li {
 line-height: 4;
}

.menu-bar {
 border: 1px solid #ebebff;
 display:flex;
 justify-content: space-between;
 padding: 20px 10px;
}



.menu-bar .input{
 width: 300px;
}

.select-bar{
 display:flex;
 justify-content: space-between;
 margin: 20px 0;
 font-size:16px;
}

.list-count{
 display:flex;
}

.btn-insert{
 float:right;
}
.paging{
 display: flex;
 justify-content: center;
 align-items: center;
}
.paging li{
 padding: 0 5px;
}


--  boardDetail
.text-max{
    width:100%;
}

.table td:nth-of-type(1){
    background-color:#fafafa;
    font-weight:bold;
}

main{
    margin:0 30px;
}

.buttons ul{
    display:flex;
    justify-content: center;

}

.buttons ul li{
    margin:0 10px;
}

#preview {
    width: 40px;
    height: 40px;
}

~~~
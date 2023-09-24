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

import com.study.admin.dto.*;
import com.study.admin.enums.BoardType;
import com.study.admin.service.BoardService;
import com.study.admin.service.CategoryService;
import com.study.admin.service.FreeService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpSession;
import java.util.List;

@Controller
@RequestMapping("/admin/free")
public class FreeController {

    @Autowired
    private FreeService freeService;

    @Autowired
    private CategoryService categoryService;

    @Autowired
    private BoardService boardService;


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



}



package com.study.admin.controller;

import com.study.admin.dto.*;
import com.study.admin.enums.BoardType;
import com.study.admin.service.BoardService;
import com.study.admin.service.CategoryService;
import com.study.admin.service.NoticeService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.*;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpSession;
import java.util.List;

@Controller
@RequestMapping("/admin/notice")
public class NoticeController  {

    @Autowired
    private NoticeService noticeService;

    @Autowired
    private CategoryService categoryService;

    @Autowired
    private BoardService boardService;

    /**
     * 공지사항 조회
     * @param model
     * @return
     */
    @GetMapping("/list")
    public String getNoticeList(Model model, HttpServletRequest request,  SearchDto searchDto){
        if(searchDto.getStartDate() != null || searchDto.getEndDate() != null){
            searchDto.setStartDate(searchDto.getStartDate()+" 00:00:00");
            searchDto.setEndDate(searchDto.getEndDate()+" 23:59:59");
        }

        BoardDto boardDto = new BoardDto();
        boardDto.setBoardType("NT");
        searchDto.setBoardType("NT");

        int totalCnt = boardService.getSearchTotalCnt(searchDto);

        List<BoardDto> noticeList = boardService.getNoticeList(searchDto);

        PageHandler pageHandler = new PageHandler(totalCnt, searchDto);




        List<BoardDto> alarmList = noticeService.getAlarmsList();

        // 로그인
        HttpSession session = request.getSession();
        AdminDto adminDto = (AdminDto) session.getAttribute("loginUser");


        //카테고리 조회
        List<CategoryDto> categoryList = categoryService.getCategoryList("NT");
        String type = "notice";

        model.addAttribute("alarmList", alarmList);
        model.addAttribute("searchDto", searchDto);
        model.addAttribute("categoryList",categoryList);
        model.addAttribute("admin",adminDto);
        model.addAttribute("board",noticeList);
        model.addAttribute("ph",pageHandler);
        model.addAttribute("type",type);

        return "board";
    }

    /**
     *  등록 페이지 조회
     * @return
     */
    @GetMapping("/view")
    public String getView(Model model, HttpServletRequest request){

        //카테고리 조회
        List<CategoryDto> categoryList = categoryService.getCategoryList("NT");



        HttpSession session = request.getSession();
        AdminDto adminDto = (AdminDto) session.getAttribute("loginUser");

        String type = "notice";


        model.addAttribute("admin",adminDto);
        model.addAttribute("categoryList",categoryList);
        model.addAttribute("formType","insert");
        model.addAttribute("type",type);
        return "boardDetail";
    }

    /**
     * 수정 페이지 조회
     * @param seq
     * @param model
     * @return
     */
    @GetMapping("/view/{seq}")
    public String getWriteForm(@PathVariable("seq") int seq, Model model){

        BoardDto notice = noticeService.getNoticeForm(seq);
        List<CategoryDto> categoryList = categoryService.getCategoryList("NT");
        noticeService.addViewCnt(seq);

        String type = "notice";

        model.addAttribute("categoryList",categoryList);
        model.addAttribute("board",notice);
        model.addAttribute("formType","update");
        model.addAttribute("type",type);
        return "boardDetail";
    }

    /**
     * 공지사항 등록
     * @param boardDto
     * @return
     */
    @PostMapping(value = "/write")
    public String addWriteForm(BoardDto boardDto)
    {
        boardDto.setBoardType("NT");
        noticeService.addWriteForm(boardDto);

        return "redirect:/admin/notice/list";
    }



    /**
     * 공지사항 수정
     * @param seq
     * @param boardDto
     * @return
     */
    @PostMapping(value = "/modify/{seq}")
    public String modifyNoticeForm(@PathVariable("seq") int seq, BoardDto boardDto)
    {
        boardDto.setBoardId(seq);
        boardDto.setTitle(boardDto.getTitle());
        boardDto.setContent(boardDto.getContent());
        boardDto.setCategoryId(boardDto.getCategoryId());

        noticeService.modifyNotice(boardDto);

        return "redirect:/admin/notice/list";
    }


    /**
     * 공지사항 삭제
     * @param seq
     * @return
     */
    @RequestMapping("/delete/{seq}")
    public String removeNotice(@PathVariable("seq") int seq){

        noticeService.removeNotice(seq);

        return "redirect:/admin/notice/list";

    }





}


~~~

## dto

~~~
package com.study.admin.dto;

import lombok.Data;

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




}


package com.study.admin.dto;

import lombok.Data;

@Data
public class SearchDto {

    private Integer page = 1;

    private String startDate;

    private String endDate;

    private String boardType; // 게시판 타입

    private String searchField;

    private String searchWord;

    private Integer limitCount = 10;

    private String searchSort;

    private String sort;


    public Integer getStartRow(){
        return (this.page-1) * this.limitCount;
    }



}


~~~

## mapper

~~~
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


    int selectTotalCnt(SearchDto searchDto);
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
public interface NoticeMapper {


    /**
     * 공지사항 폼 조회
     * @param seq
     * @return
     */
    BoardDto selectNoticeForm(int seq);

    /**
     * 공지사항 수정
     * @param boardDto
     */
    void updateNotice(BoardDto boardDto);


    /**
     * 공지사항 등록
     * @param boardDto
     */
    void insertNotice(BoardDto boardDto);

    /**
     * 공지사항 삭제
     * @param seq
     */
    void deleteNotice(int seq);

    /**
     * 공지사항 업데이트
     * @param id
     */
    void updateViewCnt(int id);




    List<BoardDto> selectAlarmList();
}


~~~


## service


~~~
package com.study.admin.service;

import com.study.admin.dto.BoardDto;
import com.study.admin.dto.SearchDto;
import com.study.admin.mapper.BoardMapper;
import com.study.admin.mapper.NoticeMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class BoardService {

    @Autowired
    private BoardMapper boardMapper;


    /**
     * 공지사항 조회
     * @return
     */
    public List<BoardDto> getNoticeList(SearchDto searchDto) { return boardMapper.selectNoticeList(searchDto);}


    public int getSearchTotalCnt(SearchDto searchDto) {return  boardMapper.selectTotalCnt(searchDto);}
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

package com.study.admin.service;

import com.study.admin.dto.BoardDto;
import com.study.admin.dto.SearchDto;
import com.study.admin.mapper.NoticeMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class NoticeService {

    @Autowired
    private NoticeMapper noticeMapper;


    /**
     * 공지사항 폼 조회
     * @param seq
     * @return
     */
    public BoardDto getNoticeForm(int seq) {return  noticeMapper.selectNoticeForm(seq);}

    /**
     * 공지사항 삭제
     * @param seq
     */
    public void removeNotice(int seq) { noticeMapper.deleteNotice(seq);}

    /**
     * 공지사항 수정
     * @param boardDto
     */
    public void modifyNotice(BoardDto boardDto) {noticeMapper.updateNotice(boardDto);}


    /**
     * 공지사항 추가
     * @param boardDto
     */
    public void addWriteForm(BoardDto boardDto) {noticeMapper.insertNotice(boardDto);}


    public void addViewCnt(int id) {noticeMapper.updateViewCnt(id);}



    public List<BoardDto>  getAlarmsList() {return noticeMapper.selectAlarmList(); }

}





~~~

##mapper

~~~
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.study.admin.mapper.BoardMapper">

    <select id="selectTotalCnt" resultType="int" parameterType="com.study.admin.dto.SearchDto">
        SELECT count(*)
        FROM  board b
        LEFT JOIN category c
        ON c.category_id = b.category_id
        WHERE 1=1
        <include refid="searchCondition"/>
    </select>

    <select id="selectNoticeList" resultType="com.study.admin.dto.BoardDto" parameterType="com.study.admin.dto.SearchDto">
        SELECT b.*,c.name as category_name
        FROM  board b
        LEFT JOIN category c
        ON c.category_id = b.category_id
        WHERE 1=1

        <include refid="searchCondition"/>
        <if test="limitCount != null and  !''.equals(limitCount)">
            LIMIT #{limitCount} OFFSET #{startRow};
        </if>
    </select>


    <sql id="searchCondition">
        <if test="boardType == 'NT'">
            AND b.board_type = 'NT'
            AND b.category_id = 1
        </if>
        <if test="boardType == 'FR'">
            AND b.board_type = 'FR'
            AND( b.category_id = 3
            OR b.category_id = 4
            OR b.category_id = 5
            )
        </if>

        <if test="startDate != null and  !''.equals(startDate)">
            AND b.created_date <![CDATA[>=]]> #{startDate}
        </if>
        <if test="endDate != null and  !''.equals(endDate)">
            AND b.created_date <![CDATA[<=]]> #{endDate}
        </if>
        <if test=" searchField != null and  !''.equals(searchField)">
            <if test="searchField != 'all'">
                AND b.category_id = #{searchField}
            </if>
        </if>
        <if test="searchWord != null and  !''.equals(searchWord)">
            AND (b.title LIKE CONCAT('%',#{searchWord},'%')
            OR b.content LIKE CONCAT('%',#{searchWord},'%'))
        </if>
        ORDER BY
        <choose>
            <when test="searchSort != null and  !''.equals(searchSort)">
                <if test="searchSort.equals('createdDate')">
                    b.created_date
                </if>
                <if test="searchSort.equals('category')">
                    b.category_id
                </if>
                <if test="searchSort.equals('title')">
                    b.title
                </if>
                <if test="searchSort.equals('viewCnt')">
                    b.view_cnt
                </if>
            </when>
            <otherwise>
                b.created_date
            </otherwise>
        </choose>


        <choose>
            <when test="sort != null and  !''.equals(sort)">
                ${sort}
            </when>
            <otherwise>
                desc
            </otherwise>
        </choose>

    </sql>

</mapper>


<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.study.admin.mapper.FreeMapper">


</mapper>


<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.study.admin.mapper.NoticeMapper">




    <select id="selectAlarmList" resultType="com.study.admin.dto.BoardDto" >
        SELECT b.*,c.name as category_name
         FROM  board b
        LEFT JOIN category c
         ON c.category_id = b.category_id
         WHERE 1=1
           AND b.board_type = 'NT'
           AND b.category_id = 2
           AND b.alarm = 'Y'
         ORDER BY created_date DESC
             LIMIT 5
    </select>




    <select id="selectNoticeForm" parameterType="int" resultType="com.study.admin.dto.BoardDto">
        SELECT *
        FROM board
        WHERE board_id = #{seq}
    </select>

    <update id="updateNotice" parameterType="com.study.admin.dto.BoardDto">
        UPDATE board SET title = #{title} , content = #{content}, category_id = #{categoryId}, alarm = #{alarm}
        WHERE board_id = #{boardId}
    </update>



    <update id="updateViewCnt" parameterType="int">
        UPDATE board SET view_cnt = view_cnt + 1
        WHERE board_id = #{id}
    </update>

    <insert id="insertNotice" parameterType="com.study.admin.dto.BoardDto">
        INSERT INTO board
        (category_id,admin_id,title,content, board_type,name,alarm,view_cnt,created_date )
        VALUES
        (#{categoryId},#{adminId},#{title},#{content},#{boardType},#{name},#{alarm},1,now())

    </insert>


    <delete id="deleteNotice"  parameterType="int">
        DELETE FROM board
        WHERE board_id = #{seq}
    </delete>
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
             <li>갤러리 게시판 관리</li>
             <li>문의 게시판 관리</li>
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
                     <select  class="searchField" name="searchField">
                        <option value="all">전체 분류</option>
                        <th:block th:each="category: ${categoryList}">
                            <option  th:selected="${searchDto.searchField}==${category.categoryId}" th:value="${category.categoryId}" th:text="${category.name}"></option>
                        </th:block>

                    </select>
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
         <div class="btn-insert">
             <a class="btn btn-default" th:href="@{/admin/{type}/view(type=${type})}" >글 등록</a>
         </div>
         <div th:replace="table :: table"></div>
         <div th:replace="page :: page"></div>

     </div>
 </main>
</body>
</html>

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

   <form method="post" id="noticeForm" th:action="${formType == 'update' ? '/admin/'+type+'/modify/' + board.boardId : '/admin/'+type+'/write'}"  t>
     <table class="table">
       <th:block th:if="${formType == 'insert'}">
         <input type="hidden" name="adminId" th:value="${admin.adminId}"/>
         <input type="hidden" name="name" th:value="${admin.name}"/>
       </th:block>

       <tbody>
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
         <tr>
           <td>제목</td>
           <td><input type="text" class="text-max" name="title" th:value="${formType == 'update'} ? ${board.title} : '' "></td>
         </tr>
         <tr>
           <td>내용</td>
           <td><textarea class="text-max" name="content" th:value="${formType == 'update'} ? ${board.content} : '' "    th:text="${formType == 'update'} ? ${board.content} : '' "></textarea></td>
         </tr>
         <th:block th:if="${type == 'notice'}">
         <tr>
           <td>알림글</td>
           <td><input type="checkbox" name="alarm" onchange="YnCheck(this);" th:checked="${formType == 'update'} ? (${board.alarm} == 'Y') : false"   ></td>
         </tr>
         </th:block>
<!--         <th:block th:if="${type == 'free'}">-->
           <td>첨부파일</td>
           <td>
               <div class="file_list">

                 <div class="file_input">
                   <input type="text" readonly />
                   <label>
                     <input type="file" name="files" onchange="selectFile(this);" />

                   </label>
                   <button type="button" onclick="removeFile(this);" class="btns del_btn"><span>x</span></button>
                 </div>
               </div>

                 <button type="button" onclick="addFile();" class="btns fn_add_btn"><span>추가</span></button>

           </td>
<!--         </th:block>-->


       </tbody>
     </table>

<!--     <th:block th:if="${type == 'free'}">-->
     <div class="container">
         <div class="row">
           <table class="table table-striped" style="text-align: center; border: 1px solid #dddddd">
             <tbody>
             <tr>
               <td >댓글</td>
             </tr>
             <tr>

               <div class="container">
                 <div class="row">
                   <table class="table table-striped" style="text-align: center; border: 1px solid #dddddd">
                     <tbody>

                     <tr th:each="comment : ${comments}">
                       <td th:text="${comment.createdDate}"></td>
                       <td th:text="${comment.content}"></td>
                     </tr>

                     </tbody>
                   </table>
                 </div>
               </div>

             </tr>
           </table>
         </div>
       </div>
<!--     </th:block>-->
   </form>
   <div class="buttons">
     <ul>
       <li th:if="${formType == 'insert'}"><button type="button" class="btn btn-primary"   id="insertNotice"  >등록</button></li>
       <li th:unless="${formType == 'insert'}"><button type="submit" class="btn btn-primary" id="updateNotice" >수정</button></li>
       <li><button type="button" class="btn btn-default" th:onclick="|location.href='@{/admin/{type}/list(type=${type})}'|" >목록</button></li>
       <li th:if="${formType == 'update'}" ><button type="button" class="btn btn-danger" th:onclick="|location.href='@{/admin/{type}/delete/{id}(id=${board.boardId},type=${type})}'|">삭제</button></li>
     </ul>
   </div>
 </main>
</body>
</html>


<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<aside th:fragment="table">
  <table class="table">
    <thead>
    <tr>
      <th>번호</th>
      <th>분류</th>
      <th>제목</th>
      <th>조회</th>
      <th>등록일시</th>
      <th>등록자</th>
    </tr>
    </thead>
    <th:block th:if="${alarmList != null}">
      <tbody>
      <tr th:each="alarm : ${alarmList}">
        <td></td>
        <td th:text="${alarm.categoryName}"></td>
        <td ><a th:text="${alarm.title}" th:onclick="|location.href='@{/admin/{type}/view/{id}(id=${alarm.boardId},type=${type})}'|" ></a></td>
        <td th:text="${alarm.viewCnt}"></td>
        <td th:text="${alarm.createdDate}"></td>
        <td th:text="${alarm.name}"></td>
      </tr>
      </tbody>
    </th:block>
    <tfoot>
    <tr th:each="board, loop : ${board}">
      <td th:text="${ph.totalCnt - ((ph.sc.page -1) * ph.sc.limitCount) -loop.index }"></td>
      <td th:text="${board.categoryName}"></td>
      <td ><a th:text="${board.title}" th:onclick="|location.href='@{/admin/{type}/view/{id}(id=${board.boardId},type=${type})}'|" ></a></td>
      <td th:text="${board.viewCnt}"></td>
      <td th:text="${board.createdDate}"></td>
      <td th:text="${board.name}"></td>
    </tr>
    </tfoot>


  </table>
</aside>

<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<footer th:fragment="page">
    <div class="paging-container">
        <div class="paging" >
            <th:block th:if="${ph.totalCnt == null or ph.totalCnt == 0}">
                <div>게시물이 없습니다.</div>
            </th:block>
            <th:block th:if="${ph.totalCnt != null and ph.totalCnt != 0}">
                <li th:if="${ph.showPrev}">
                    <a  th:onclick="'pageForms(' + (${ph.beginPage} - 1)+')'" >&lt;</a>
                </li>

                <li th:each="page: ${#numbers.sequence(ph.beginPage,ph.endPage)}"class="page-item">
                    <a th:text="${page}" th:onclick="'pageForms(' + ${page}  +')'" ></a>
                </li>

                <li th:if="${ph.showNext}">
                    <a  th:onclick="'pageForms(' + (${ph.endPage} + 1) +  ')'" >&gt;</a>
                </li>
            </th:block>

        </div>
    </div>
</footer>
</html>

~~~


## js

~~~
// board
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

//boardDetail
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


// 파일 추가
function addFile() {
    const fileDiv = document.createElement('div');
    fileDiv.innerHTML =`
          <div class="file_input">
                   <input type="text" readonly />
                   <label>
                     <input type="file" name="files" onchange="selectFile(this);" />
                  
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




~~~
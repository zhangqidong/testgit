package org.mspring.mlog;

import com.alibaba.fastjson.JSON;
import org.mspring.mlog.dao.ArticleDao;
import org.mspring.mlog.dao.CommentDao;
import org.mspring.mlog.entity.Article;
import org.mspring.mlog.entity.Comment;
import org.mspring.mlog.entity.em.ArticleStatus;
import org.mspring.nbee.common.utils.CollectionUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

/**
 * 搜狐畅言
 *
 * @author Gao Youbo
 * @since 2015-01-26 13:16
 */
@Component
public class ChangyanService {
    private ArticleDao articleDao;
    private CommentDao commentDao;

    /**
     * 生成导入所需要的JSON
     *
     * @return
     */
    public String generateImportJson() {
        List<Article> articleList = articleDao.listAll();
        if (CollectionUtils.isEmpty(articleList)) {
            return null;
        }
        StringBuilder sb = new StringBuilder();
        for (Article article : articleList) {
            if (article.getStatus() != ArticleStatus.NORMAL) {
                continue;
            }
            List<Comment> commentList = commentDao.listByArticle(article.getId());
            if (CollectionUtils.isEmpty(commentList)) {
                continue;
            }
            List<Map<String, Object>> comments = new ArrayList<>();
            for (Comment comment : commentList) {
                Map<String, Object> user = new HashMap<>();
                user.put("userid", 1);
                user.put("nickname", comment.getAuthorName());
                user.put("usericon", comment.getAuthorAvatar());

                Map<String, Object> commentMap = new HashMap<>();
                commentMap.put("cmtid", comment.getId());
                commentMap.put("ctime", comment.getCreateTime().getTime());
                commentMap.put("content", comment.getContent());
                commentMap.put("replyid", comment.getReplyId() == null ? 0L : comment.getReplyId());
                commentMap.put("ip", comment.getIp());
                commentMap.put("useragent", comment.getAgent());
                commentMap.put("channeltype", "1");
                commentMap.put("from", "");
                commentMap.put("spcount", "");
                commentMap.put("opcount", "");
                commentMap.put("user", user);
                comments.add(commentMap);
            }

            Map<String, Object> item = new HashMap<>();
            item.put("title", article.getTitle());
            item.put("url", "");
            item.put("ttime", article.getCreateTime().getTime());
            item.put("sourceid", article.getId());
            item.put("parentid", "");
            item.put("categoryid", article.getCategory());
            item.put("ownerid", article.getCreateUser());
            item.put("metadata", "");
            item.put("comments", comments);

            sb.append(JSON.toJSONString(item)).append("\n");
        }
        return sb.toString();
    }

    @Autowired
    public void setArticleDao(ArticleDao articleDao) {
        this.articleDao = articleDao;
    }

    @Autowired
    public void setCommentDao(CommentDao commentDao) {
        this.commentDao = commentDao;
    }
}


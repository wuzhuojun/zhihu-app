# Laravel Vuejs 实战视频：开发知乎

视频教程：https://www.laravist.com/series/build-a-zhihu-website-with-laravel

## 数据模型（Models）

本项目使用 Laravel Eloquent ORM，包含以下数据模型：

### User（用户）
- **文件**：`app/User.php`
- **表名**：`users`
- **字段**：`name`, `email`, `password`, `avatar`, `confirmation_token`, `api_token`, `settings`
- **关联关系**：
  - `answers()` — 拥有多个回答（hasMany Answer）
  - `follows()` — 关注的问题（belongsToMany Question，通过 `user_question` 中间表）
  - `followers()` — 关注的用户（belongsToMany User，通过 `followers` 表）
  - `followersUser()` — 关注自己的用户（belongsToMany User，通过 `followers` 表）
  - `votes()` — 赞同的回答（belongsToMany Answer，通过 `votes` 表）
  - `messages()` — 收到的私信（hasMany Message）

### Question（问题）
- **文件**：`app/Question.php`
- **表名**：`questions`
- **字段**：`title`, `body`, `user_id`, `is_hidden`, `close_comment`
- **关联关系**：
  - `user()` — 所属用户（belongsTo User）
  - `topics()` — 所属话题（belongsToMany Topic）
  - `answers()` — 拥有多个回答（hasMany Answer）
  - `followers()` — 关注该问题的用户（belongsToMany User，通过 `user_question` 表）
  - `comments()` — 评论（morphMany Comment）

### Answer（回答）
- **文件**：`app/Answer.php`
- **表名**：`answers`
- **字段**：`user_id`, `question_id`, `body`, `votes_count`, `comments_count`
- **关联关系**：
  - `user()` — 所属用户（belongsTo User）
  - `question()` — 所属问题（belongsTo Question）
  - `comments()` — 评论（morphMany Comment）

### Topic（话题）
- **文件**：`app/Topic.php`
- **表名**：`topics`
- **字段**：`name`, `bio`, `questions_count`
- **关联关系**：
  - `questions()` — 该话题下的问题（belongsToMany Question）

### Comment（评论）
- **文件**：`app/Comment.php`
- **表名**：`comments`
- **字段**：`user_id`, `body`, `commentable_id`, `commentable_type`
- **关联关系**：
  - `commentable()` — 多态关联（morphTo，可关联到 Question 或 Answer）
  - `user()` — 所属用户（belongsTo User）

### Message（私信）
- **文件**：`app/Message.php`
- **表名**：`messages`
- **字段**：`from_user_id`, `to_user_id`, `body`, `dialog_id`, `has_read`, `read_at`
- **关联关系**：
  - `fromUser()` — 发送者（belongsTo User）
  - `toUser()` — 接收者（belongsTo User）

### Vote（赞同/反对）
- **文件**：`app/Vote.php`
- **表名**：`votes`
- 作为 User 与 Answer 之间多对多关系的中间表

### Follow（关注问题）
- **文件**：`app/Follow.php`
- **表名**：`user_question`
- **字段**：`user_id`, `question_id`
- 作为 User 与 Question 之间多对多关系的中间表

### Setting（用户设置）
- **文件**：`app/Setting.php`
- 用户个性化配置，以 JSON 格式存储在 `users` 表的 `settings` 字段中

### MessageCollection（私信集合）
- **文件**：`app/MessageCollection.php`
- 自定义 Eloquent Collection 类，用于批量处理 Message 模型集合

## 模型关系总览

```
User ──── hasMany ──────────────────► Answer
User ──── belongsToMany(Question) ──► user_question（关注问题）
User ──── belongsToMany(User) ──────► followers（互相关注）
User ──── belongsToMany(Answer) ────► votes（赞同回答）
User ──── hasMany ──────────────────► Message

Question ── belongsTo ──────────────► User
Question ── hasMany ────────────────► Answer
Question ── belongsToMany(Topic) ───► question_topic
Question ── belongsToMany(User) ────► user_question（被关注）
Question ── morphMany ──────────────► Comment

Answer ──── belongsTo ──────────────► User
Answer ──── belongsTo ──────────────► Question
Answer ──── morphMany ──────────────► Comment

Topic ───── belongsToMany(Question) ► question_topic

Comment ─── morphTo ────────────────► Question / Answer
Comment ─── belongsTo ──────────────► User

Message ─── belongsTo ──────────────► User (from_user_id)
Message ─── belongsTo ──────────────► User (to_user_id)
```
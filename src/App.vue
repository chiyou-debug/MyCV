<template>
  <div class="cv-container">
    <!-- 编辑/预览切换按钮 -->
    <div class="edit-toggle no-print">
      <el-button
        :type="isEditMode ? 'primary' : 'default'"
        @click="isEditMode = !isEditMode"
        :icon="isEditMode ? View : Edit"
      >
        {{ isEditMode ? "预览模式" : "编辑模式" }}
      </el-button>
      <el-button
        type="success"
        @click="exportPDF"
        :icon="Download"
        :loading="isExporting"
        style="margin-left: 10px"
      >
        导出PDF
      </el-button>
      <el-button
        :type="showPageBreaks ? 'warning' : 'default'"
        @click="showPageBreaks = !showPageBreaks"
        style="margin-left: 10px"
      >
        {{ showPageBreaks ? "隐藏分页线" : "显示分页线" }}
      </el-button>
    </div>

    <!-- 简历内容区域（用于导出PDF） -->
    <div
      ref="cvContent"
      class="cv-content"
      :class="{ 'show-page-breaks': showPageBreaks }"
    >
      <!-- 分页线标记 -->
      <div v-if="showPageBreaks" class="page-break-lines">
        <div
          v-for="n in 10"
          :key="n"
          class="page-break-line"
          :style="{ top: n * a4HeightInPixels + 'px' }"
        >
          <span class="page-break-label">第 {{ n + 1 }} 页开始</span>
        </div>
      </div>
      <!-- 编辑面板 -->
      <div v-if="isEditMode" class="edit-panel">
        <el-tabs type="border-card">
          <!-- 基本信息 -->
          <el-tab-pane label="基本信息">
            <el-form :model="resumeData.basic" label-width="100px">
              <el-form-item label="头像">
                <div class="avatar-edit">
                  <el-avatar :size="80" :src="resumeData.basic.avatar" />
                  <el-input
                    v-model="resumeData.basic.avatar"
                    placeholder="头像图片URL"
                    style="width: 300px; margin-left: 10px"
                  />
                  <el-upload
                    class="avatar-uploader"
                    action="#"
                    :auto-upload="false"
                    :show-file-list="false"
                    :on-change="handleAvatarChange"
                    accept="image/*"
                  >
                    <el-button type="primary" size="small">选择图片</el-button>
                  </el-upload>
                </div>
              </el-form-item>
              <el-form-item label="姓名">
                <el-input v-model="resumeData.basic.name" />
              </el-form-item>
              <el-form-item label="职位">
                <el-input v-model="resumeData.basic.title" />
              </el-form-item>
              <el-form-item label="副标题">
                <el-input v-model="resumeData.basic.subtitle" />
              </el-form-item>
              <el-form-item label="邮箱">
                <el-input v-model="resumeData.basic.email" />
              </el-form-item>
              <el-form-item label="电话">
                <el-input v-model="resumeData.basic.phone" />
              </el-form-item>
              <el-form-item label="地址">
                <el-input v-model="resumeData.basic.location" />
              </el-form-item>
              <el-form-item label="GitHub">
                <el-input v-model="resumeData.basic.github" />
              </el-form-item>
            </el-form>
          </el-tab-pane>

          <!-- 个人简介 -->
          <el-tab-pane label="个人简介">
            <el-form :model="resumeData" label-width="100px">
              <el-form-item label="简介内容">
                <el-input
                  v-model="resumeData.introduction"
                  type="textarea"
                  :rows="10"
                  placeholder="输入个人简介，用换行分隔段落"
                />
              </el-form-item>
            </el-form>
          </el-tab-pane>

          <!-- 技术栈 -->
          <el-tab-pane label="技术栈">
            <div
              v-for="(category, cIndex) in resumeData.skillCategories"
              :key="category.key"
              class="edit-section"
              draggable="true"
              @dragstart="handleDragStart(cIndex)"
              @dragover.prevent="handleSkillDragOver(cIndex)"
              @drop="handleDrop(cIndex)"
              @dragend="handleDragEnd"
              @dragenter.prevent="handleSkillDragEnter(cIndex)"
              :class="{
                dragging: dragIndex === cIndex,
                'drag-over': dragOverIndex === cIndex && dragIndex !== cIndex,
              }"
            >
              <!-- 拖拽插入指示器 - 上方 -->
              <div
                v-if="
                  dragOverIndex === cIndex &&
                  dragIndex !== cIndex &&
                  dragIndex !== cIndex - 1
                "
                class="drop-indicator drop-indicator-top"
              >
                <div class="drop-indicator-line"></div>
                <div class="drop-indicator-label">插入到此处</div>
              </div>
              <div class="section-header">
                <div class="drag-handle">
                  <el-icon><Rank /></el-icon>
                  <el-input
                    v-model="category.title"
                    placeholder="分类标题"
                    style="width: 200px"
                  />
                </div>
                <el-button
                  type="danger"
                  size="small"
                  @click="removeSkillCategory(cIndex)"
                  >删除分类</el-button
                >
              </div>
              <div class="tag-editor">
                <el-tag
                  v-for="(skill, index) in resumeData[category.key]"
                  :key="index"
                  closable
                  @close="removeSkill(category.key, index)"
                  class="edit-tag"
                >
                  {{ skill }}
                </el-tag>
                <el-input
                  v-model="skillInputValues[category.key]"
                  placeholder="添加技能"
                  size="small"
                  @keyup.enter="addSkillFromInput(category.key)"
                  style="width: 120px"
                />
              </div>
              <!-- 拖拽插入指示器 - 下方（最后一个元素） -->
              <div
                v-if="
                  cIndex === resumeData.skillCategories.length - 1 &&
                  dragOverIndex === cIndex + 1 &&
                  dragIndex !== cIndex
                "
                class="drop-indicator drop-indicator-bottom"
              >
                <div class="drop-indicator-line"></div>
                <div class="drop-indicator-label">插入到此处</div>
              </div>
            </div>
            <el-button type="primary" @click="addSkillCategory"
              >添加技能分类</el-button
            >
            <div class="edit-section">
              <h4>技能熟练度</h4>
              <div
                v-for="(item, index) in resumeData.skillProgress"
                :key="index"
                class="skill-edit-item"
              >
                <el-input
                  v-model="item.name"
                  placeholder="技能名称"
                  size="small"
                  style="width: 150px"
                />
                <el-slider
                  v-model="item.percent"
                  :max="100"
                  style="width: 200px; margin: 0 10px"
                />
                <span>{{ item.percent }}%</span>
              </div>
            </div>
          </el-tab-pane>

          <!-- 教育背景 -->
          <el-tab-pane label="教育背景">
            <div
              v-for="(edu, index) in resumeData.education"
              :key="index"
              class="edit-section"
              draggable="true"
              @dragstart="handleEduDragStart(index)"
              @dragover.prevent="handleEduDragOver(index)"
              @drop="handleEduDrop(index)"
              @dragend="handleEduDragEnd"
              @dragenter.prevent="handleEduDragEnter(index)"
              :class="{
                dragging: eduDragIndex === index,
                'drag-over':
                  eduDragOverIndex === index && eduDragIndex !== index,
              }"
            >
              <!-- 拖拽插入指示器 - 上方 -->
              <div
                v-if="
                  eduDragOverIndex === index &&
                  eduDragIndex !== index &&
                  eduDragIndex !== index - 1
                "
                class="drop-indicator drop-indicator-top"
              >
                <div class="drop-indicator-line"></div>
                <div class="drop-indicator-label">插入到此处</div>
              </div>
              <div class="section-header">
                <h4>
                  <el-icon class="drag-handle"><Rank /></el-icon>
                  教育经历 {{ index + 1 }}
                </h4>
                <el-button
                  type="danger"
                  size="small"
                  @click="removeEducation(index)"
                  >删除</el-button
                >
              </div>
              <el-form label-width="100px">
                <el-form-item label="学校">
                  <el-input v-model="edu.school" />
                </el-form-item>
                <el-form-item label="学历">
                  <el-input v-model="edu.degree" />
                </el-form-item>
                <el-form-item label="专业">
                  <el-input v-model="edu.major" />
                </el-form-item>
                <el-form-item label="时间">
                  <el-input v-model="edu.time" />
                </el-form-item>
                <el-form-item label="GPA">
                  <el-input v-model="edu.gpa" />
                </el-form-item>
                <el-form-item label="排名">
                  <el-input v-model="edu.rank" />
                </el-form-item>
                <el-form-item label="描述">
                  <el-input
                    v-model="edu.description"
                    type="textarea"
                    :rows="3"
                  />
                </el-form-item>
              </el-form>
              <!-- 拖拽插入指示器 - 下方（最后一个元素） -->
              <div
                v-if="
                  index === resumeData.education.length - 1 &&
                  eduDragOverIndex === index + 1 &&
                  eduDragIndex !== index
                "
                class="drop-indicator drop-indicator-bottom"
              >
                <div class="drop-indicator-line"></div>
                <div class="drop-indicator-label">插入到此处</div>
              </div>
            </div>
            <el-button type="primary" @click="addEducation"
              >添加教育经历</el-button
            >
          </el-tab-pane>

          <!-- 证书荣誉 -->
          <el-tab-pane label="证书荣誉">
            <div
              v-for="(cert, index) in resumeData.certificates"
              :key="index"
              class="edit-section"
              draggable="true"
              @dragstart="handleCertDragStart(index)"
              @dragover.prevent="handleCertDragOver(index)"
              @drop="handleCertDrop(index)"
              @dragend="handleCertDragEnd"
              @dragenter.prevent="handleCertDragEnter(index)"
              :class="{
                dragging: certDragIndex === index,
                'drag-over':
                  certDragOverIndex === index && certDragIndex !== index,
              }"
            >
              <!-- 拖拽插入指示器 - 上方 -->
              <div
                v-if="
                  certDragOverIndex === index &&
                  certDragIndex !== index &&
                  certDragIndex !== index - 1
                "
                class="drop-indicator drop-indicator-top"
              >
                <div class="drop-indicator-line"></div>
                <div class="drop-indicator-label">插入到此处</div>
              </div>
              <div class="section-header">
                <h4>
                  <el-icon class="drag-handle"><Rank /></el-icon>
                  证书 {{ index + 1 }}
                </h4>
                <el-button
                  type="danger"
                  size="small"
                  @click="removeCertificate(index)"
                  >删除</el-button
                >
              </div>
              <el-form label-width="100px">
                <el-form-item label="证书名称">
                  <el-input v-model="cert.name" />
                </el-form-item>
                <el-form-item label="获得时间">
                  <el-input v-model="cert.time" />
                </el-form-item>
              </el-form>
              <!-- 拖拽插入指示器 - 下方（最后一个元素） -->
              <div
                v-if="
                  index === resumeData.certificates.length - 1 &&
                  certDragOverIndex === index + 1 &&
                  certDragIndex !== index
                "
                class="drop-indicator drop-indicator-bottom"
              >
                <div class="drop-indicator-line"></div>
                <div class="drop-indicator-label">插入到此处</div>
              </div>
            </div>
            <el-button type="primary" @click="addCertificate"
              >添加证书</el-button
            >
          </el-tab-pane>

          <!-- 工作经历 -->
          <el-tab-pane label="工作经历">
            <div
              v-for="(work, index) in resumeData.workExperience"
              :key="index"
              class="edit-section"
              draggable="true"
              @dragstart="handleWorkDragStart(index)"
              @dragover.prevent="handleWorkDragOver(index)"
              @drop="handleWorkDrop(index)"
              @dragend="handleWorkDragEnd"
              @dragenter.prevent="handleWorkDragEnter(index)"
              :class="{
                dragging: workDragIndex === index,
                'drag-over':
                  workDragOverIndex === index && workDragIndex !== index,
              }"
            >
              <!-- 拖拽插入指示器 - 上方 -->
              <div
                v-if="
                  workDragOverIndex === index &&
                  workDragIndex !== index &&
                  workDragIndex !== index - 1
                "
                class="drop-indicator drop-indicator-top"
              >
                <div class="drop-indicator-line"></div>
                <div class="drop-indicator-label">插入到此处</div>
              </div>
              <div class="section-header">
                <h4>
                  <el-icon class="drag-handle"><Rank /></el-icon>
                  工作经历 {{ index + 1 }}
                </h4>
                <el-button type="danger" size="small" @click="removeWork(index)"
                  >删除</el-button
                >
              </div>
              <el-form label-width="100px">
                <el-form-item label="公司">
                  <el-input v-model="work.company" />
                </el-form-item>
                <el-form-item label="职位">
                  <el-input v-model="work.position" />
                </el-form-item>
                <el-form-item label="部门">
                  <el-input v-model="work.department" />
                </el-form-item>
                <el-form-item label="时间">
                  <el-input v-model="work.time" />
                </el-form-item>
                <el-form-item label="工作业绩">
                  <div
                    v-for="(achievement, aIndex) in work.achievements"
                    :key="aIndex"
                    class="achievement-item"
                  >
                    <el-input
                      v-model="work.achievements[aIndex]"
                      type="textarea"
                      :rows="2"
                    />
                    <el-button
                      type="danger"
                      size="small"
                      @click="removeAchievement(index, aIndex)"
                      >删除</el-button
                    >
                  </div>
                  <el-button
                    type="primary"
                    size="small"
                    @click="addAchievement(index)"
                    >添加业绩</el-button
                  >
                </el-form-item>
                <el-form-item label="核心技术">
                  <div class="tag-editor">
                    <el-tag
                      v-for="(tech, tIndex) in work.techStack"
                      :key="tIndex"
                      closable
                      @close="work.techStack.splice(tIndex, 1)"
                      class="edit-tag"
                    >
                      {{ tech }}
                    </el-tag>
                    <el-input
                      placeholder="添加技术"
                      size="small"
                      @keyup.enter="addWorkTech(index, $event)"
                      style="width: 120px"
                    />
                  </div>
                </el-form-item>
              </el-form>
              <!-- 拖拽插入指示器 - 下方（最后一个元素） -->
              <div
                v-if="
                  index === resumeData.workExperience.length - 1 &&
                  workDragOverIndex === index + 1 &&
                  workDragIndex !== index
                "
                class="drop-indicator drop-indicator-bottom"
              >
                <div class="drop-indicator-line"></div>
                <div class="drop-indicator-label">插入到此处</div>
              </div>
            </div>
            <el-button type="primary" @click="addWork">添加工作经历</el-button>
          </el-tab-pane>

          <!-- 项目经验 -->
          <el-tab-pane label="项目经验">
            <div
              v-for="(project, index) in resumeData.projects"
              :key="index"
              class="edit-section"
              draggable="true"
              @dragstart="handleProjectDragStart(index)"
              @dragover.prevent="handleProjectDragOver(index)"
              @drop="handleProjectDrop(index)"
              @dragend="handleProjectDragEnd"
              @dragenter.prevent="handleProjectDragEnter(index)"
              :class="{
                dragging: projectDragIndex === index,
                'drag-over':
                  projectDragOverIndex === index && projectDragIndex !== index,
              }"
            >
              <!-- 拖拽插入指示器 - 上方 -->
              <div
                v-if="
                  projectDragOverIndex === index &&
                  projectDragIndex !== index &&
                  projectDragIndex !== index - 1
                "
                class="drop-indicator drop-indicator-top"
              >
                <div class="drop-indicator-line"></div>
                <div class="drop-indicator-label">插入到此处</div>
              </div>
              <div class="section-header">
                <h4>
                  <el-icon class="drag-handle"><Rank /></el-icon>
                  项目 {{ index + 1 }}
                </h4>
                <el-button
                  type="danger"
                  size="small"
                  @click="removeProject(index)"
                  >删除</el-button
                >
              </div>
              <el-form label-width="100px">
                <el-form-item label="项目名称">
                  <el-input v-model="project.name" />
                </el-form-item>
                <el-form-item label="时间">
                  <el-input v-model="project.time" />
                </el-form-item>
                <el-form-item label="角色">
                  <el-input v-model="project.role" />
                </el-form-item>
                <el-form-item label="项目描述">
                  <el-input
                    v-model="project.description"
                    type="textarea"
                    :rows="3"
                  />
                </el-form-item>
                <el-form-item label="项目标签">
                  <div class="tag-editor">
                    <el-tag
                      v-for="(tag, tIndex) in project.tags"
                      :key="tIndex"
                      closable
                      :type="tag.type || 'primary'"
                      @close="project.tags.splice(tIndex, 1)"
                      class="edit-tag"
                    >
                      {{ tag.name }}
                    </el-tag>
                    <el-input
                      v-model="projectTagInput[index]"
                      placeholder="添加标签"
                      size="small"
                      @keyup.enter="addProjectTag(index)"
                      style="width: 120px"
                    />
                  </div>
                </el-form-item>
                <el-form-item label="技术架构">
                  <div class="tech-stack-container">
                    <div
                      v-for="(tech, key) in project.techStack"
                      :key="key"
                      class="tech-stack-item"
                    >
                      <span class="tech-key">{{ key }}：</span>
                      <el-input
                        v-model="project.techStack[key]"
                        size="small"
                        style="width: 200px"
                      />
                      <el-button
                        type="danger"
                        size="small"
                        @click="delete project.techStack[key]"
                        >删除</el-button
                      >
                    </div>
                    <div class="add-tech-stack">
                      <el-input
                        v-model="newTechKey[index]"
                        placeholder="技术类别"
                        size="small"
                        style="width: 100px"
                      />
                      <span class="tech-separator">：</span>
                      <el-input
                        v-model="newTechValue[index]"
                        placeholder="技术内容"
                        size="small"
                        style="width: 200px"
                      />
                      <el-button
                        type="primary"
                        size="small"
                        @click="addProjectTechStack(index)"
                        >添加</el-button
                      >
                    </div>
                  </div>
                </el-form-item>
                <el-form-item label="项目成果">
                  <div
                    v-for="(result, rIndex) in project.results"
                    :key="rIndex"
                    class="result-item"
                  >
                    <el-input
                      v-model="project.results[rIndex]"
                      type="textarea"
                      :rows="2"
                    />
                    <el-button
                      type="danger"
                      size="small"
                      @click="removeProjectResult(index, rIndex)"
                      >删除</el-button
                    >
                  </div>
                  <el-button
                    type="primary"
                    size="small"
                    @click="addProjectResult(index)"
                    >添加成果</el-button
                  >
                </el-form-item>
              </el-form>
              <!-- 拖拽插入指示器 - 下方（最后一个元素） -->
              <div
                v-if="
                  index === resumeData.projects.length - 1 &&
                  projectDragOverIndex === index + 1 &&
                  projectDragIndex !== index
                "
                class="drop-indicator drop-indicator-bottom"
              >
                <div class="drop-indicator-line"></div>
                <div class="drop-indicator-label">插入到此处</div>
              </div>
            </div>
            <el-button type="primary" @click="addProject">添加项目</el-button>
          </el-tab-pane>
        </el-tabs>
      </div>

      <!-- 顶部个人信息区域 -->
      <header class="header-section">
        <div class="header-content">
          <div class="avatar-section">
            <div class="avatar">
              <img
                :src="resumeData.basic.avatar"
                alt="头像"
                style="
                  width: 100px;
                  height: 100px;
                  border-radius: 50%;
                  object-fit: cover;
                "
                crossorigin="anonymous"
              />
            </div>
          </div>
          <div class="info-section">
            <div class="name-row">
              <h1 class="name">{{ resumeData.basic.name }}</h1>
              <el-tag class="title-tag" type="success" effect="dark">{{
                resumeData.basic.title
              }}</el-tag>
              <el-tag class="title-tag" type="warning" effect="dark"
                >AI Agent全栈开发经验</el-tag
              >
            </div>
            <p class="subtitle">{{ resumeData.basic.subtitle }}</p>
            <div class="contact-row">
              <span class="contact-item"
                ><el-icon><Message /></el-icon>
                {{ resumeData.basic.email }}</span
              >
              <span class="contact-item"
                ><el-icon><Phone /></el-icon> {{ resumeData.basic.phone }}</span
              >
              <span class="contact-item"
                ><el-icon><Location /></el-icon>
                {{ resumeData.basic.location }}</span
              >
              <span class="contact-item"
                ><el-icon><Link /></el-icon> {{ resumeData.basic.github }}</span
              >
            </div>
          </div>
        </div>
      </header>

      <!-- 主体内容区域 -->
      <main class="main-content">
        <!-- 左侧栏 -->
        <aside class="left-sidebar">
          <!-- 技术栈 -->
          <section class="section">
            <h2 class="section-title">
              <el-icon><Cpu /></el-icon> 技术栈
            </h2>

            <div
              class="skill-category"
              v-for="category in resumeData.skillCategories"
              :key="category.key"
            >
              <h3>{{ category.title }}</h3>
              <div class="skill-tags">
                <el-tag
                  v-for="skill in resumeData[category.key]"
                  :key="skill"
                  class="skill-tag"
                  :type="getTagType(category.type)"
                  effect="plain"
                  >{{ skill }}</el-tag
                >
              </div>
            </div>

            <div class="skill-progress">
              <div
                v-for="item in resumeData.skillProgress"
                :key="item.name"
                class="progress-item"
              >
                <div class="progress-label">
                  <span>{{ item.name }}</span>
                  <span>{{ item.percent }}%</span>
                </div>
                <el-progress
                  :percentage="item.percent"
                  :color="item.color"
                  :show-text="false"
                />
              </div>
            </div>
          </section>

          <!-- 教育背景 -->
          <section class="section">
            <h2 class="section-title">
              <el-icon><School /></el-icon> 教育背景
            </h2>
            <div
              class="education-item"
              v-for="edu in resumeData.education"
              :key="edu.school"
            >
              <div class="edu-header">
                <h3 class="school-name">{{ edu.school }}</h3>
                <span class="edu-tag">{{ edu.degree }}</span>
              </div>
              <p class="edu-major">{{ edu.major }}</p>
              <p class="edu-time">{{ edu.time }}</p>
              <p class="edu-detail">GPA: {{ edu.gpa }} | {{ edu.rank }}</p>
              <p class="edu-desc">{{ edu.description }}</p>
            </div>
          </section>

          <!-- 证书荣誉 -->
          <section class="section">
            <h2 class="section-title">
              <el-icon><Trophy /></el-icon> 证书荣誉
            </h2>
            <div class="cert-list">
              <div
                class="cert-item"
                v-for="cert in resumeData.certificates"
                :key="cert.name"
              >
                <el-icon class="cert-icon" :size="20"><Medal /></el-icon>
                <div class="cert-info">
                  <p class="cert-name">{{ cert.name }}</p>
                  <p class="cert-time">{{ cert.time }}</p>
                </div>
              </div>
            </div>
          </section>
        </aside>

        <!-- 右侧栏 -->
        <div class="right-content">
          <!-- 个人简介 -->
          <section class="section">
            <h2 class="section-title">
              <el-icon><User /></el-icon> 个人简介
            </h2>
            <div class="intro-content">
              <p v-for="(para, index) in introParagraphs" :key="index">
                {{ para }}
              </p>
            </div>
          </section>

          <!-- 工作经历 -->
          <section class="section">
            <h2 class="section-title">
              <el-icon><Briefcase /></el-icon> 工作经历
            </h2>
            <div
              class="work-item"
              v-for="work in resumeData.workExperience"
              :key="work.company"
            >
              <div class="work-header">
                <div class="company-info">
                  <h3 class="company-name">{{ work.company }}</h3>
                  <el-tag
                    class="work-position"
                    type="success"
                    effect="dark"
                    size="small"
                    >{{ work.position }}</el-tag
                  >
                </div>
                <span class="work-time">{{ work.time }}</span>
              </div>
              <p class="work-dept">{{ work.department }}</p>
              <div class="work-content">
                <template
                  v-if="work.achievements && work.achievements.length > 0"
                >
                  <h4>工作业绩：</h4>
                  <ul>
                    <li
                      v-for="(achievement, index) in work.achievements"
                      :key="index"
                    >
                      {{ achievement }}
                    </li>
                  </ul>
                </template>
                <template v-if="work.techStack && work.techStack.length > 0">
                  <h4>核心技术：</h4>
                  <div class="tech-tags">
                    <el-tag
                      v-for="tech in work.techStack"
                      :key="tech"
                      size="small"
                      effect="plain"
                      >{{ tech }}</el-tag
                    >
                  </div>
                </template>
              </div>
            </div>
          </section>

          <!-- 项目经验 -->
          <section class="section">
            <h2 class="section-title">
              <el-icon><FolderOpened /></el-icon> 项目经验
            </h2>
            <div
              class="project-item"
              v-for="project in resumeData.projects"
              :key="project.name"
            >
              <div class="project-header">
                <h3 class="project-name">{{ project.name }}</h3>
                <div class="project-tags">
                  <el-tag
                    v-for="tag in project.tags"
                    :key="tag"
                    :type="tag.type"
                    size="small"
                    effect="dark"
                    >{{ tag.name }}</el-tag
                  >
                </div>
              </div>
              <p class="project-info">
                项目时间：{{ project.time }} | 项目角色：{{ project.role }}
              </p>
              <p class="project-desc" v-if="project.description">
                {{ project.description }}
              </p>
              <div
                class="project-tech"
                v-if="
                  project.techStack && Object.keys(project.techStack).length > 0
                "
              >
                <h4>技术架构：</h4>
                <div class="tech-arch">
                  <div
                    v-for="(tech, key) in project.techStack"
                    :key="key"
                    class="tech-item"
                  >
                    <span class="tech-label">{{ key }}：</span>
                    <el-tag size="small" effect="plain">{{ tech }}</el-tag>
                  </div>
                </div>
              </div>
              <div
                class="project-result"
                v-if="project.results && project.results.length > 0"
              >
                <h4>项目成果：</h4>
                <ul>
                  <li v-for="(result, index) in project.results" :key="index">
                    {{ result }}
                  </li>
                </ul>
              </div>
            </div>
          </section>
        </div>
      </main>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, reactive } from "vue";
import { Edit, View, Download } from "@element-plus/icons-vue";
import html2canvas from "html2canvas";
import jsPDF from "jspdf";

// 编辑模式状态
const isEditMode = ref(false);

// 导出状态
const isExporting = ref(false);

// 显示分页线状态
const showPageBreaks = ref(false);

// A4纸在屏幕上的像素高度
// PDF导出时的逻辑：内容宽度填满PDF页面，高度按比例缩放
// A4纸宽度 = 210mm，高度 = 297mm
// 在屏幕上，内容区域宽度约为1200px，对应PDF的210mm
// 所以PDF的297mm对应屏幕像素 = (297/210) * 内容宽度
const a4HeightInPixels = computed(() => {
  if (!cvContent.value) return 1123; // 默认A4高度像素值

  // 获取内容区域宽度
  const contentWidth = cvContent.value.offsetWidth;

  // A4宽高比 = 210:297 ≈ 1:1.414
  // 屏幕上的分页高度 = 内容宽度 * (297/210)
  const a4Ratio = 297 / 210;
  return contentWidth * a4Ratio;
});

// 简历内容引用
const cvContent = ref(null);

// 技能输入框值
const skillInputValues = reactive({});

// 项目标签输入框值
const projectTagInput = reactive({});

// 项目技术架构新增输入框值
const newTechKey = reactive({});
const newTechValue = reactive({});

// 拖拽排序状态（技术栈分类）
const dragIndex = ref(-1);
const dragOverIndex = ref(-1);

// 工作经历拖拽排序状态
const workDragIndex = ref(-1);
const workDragOverIndex = ref(-1);

// 项目经验拖拽排序状态
const projectDragIndex = ref(-1);
const projectDragOverIndex = ref(-1);

// 证书荣誉拖拽排序状态
const certDragIndex = ref(-1);
const certDragOverIndex = ref(-1);

// 教育背景拖拽排序状态
const eduDragIndex = ref(-1);
const eduDragOverIndex = ref(-1);

// 简历数据
const resumeData = reactive({
  basic: {
    name: "帅哥",
    title: "高级Java后端开发工程师",
    subtitle: "6年Java后端开发经验 | 擅长分布式系统设计与AI应用开发",
    email: "liqiang@example.com",
    phone: "135-9876-5432",
    location: "上海市浦东新区",
    github: "github.com/liqiang",
    avatar:
      "https://cube.elemecdn.com/0/88/03b0d39583f48206768a7534e55bcpng.png",
  },
  introduction: `6年Java后端开发经验，精通Java生态技术栈，具备丰富的分布式系统设计和高并发系统开发经验。擅长从0到1搭建系统架构，主导过多个亿级流量系统的设计与开发。近年来专注于AI技术落地，具备AI Agent全栈开发经验，能够独立完成从后端服务到前端展示的完整AI应用开发。

具备良好的技术视野和创新思维，热衷于探索新技术，在云原生、大模型应用开发等领域有深入研究。曾主导研发的企业级AI Agent平台，帮助公司实现业务流程自动化，提升效率300%，创造千万级业务价值。希望能够加入优秀团队，利用自身技术经验，为公司创造更大价值。`,
  skillCategories: [
    { key: "JavaSkills", title: "JavaSE", type: "success" },
    { key: "JavaWebSkills", title: "JavaWeb", type: "warning" },
    { key: "SpringCloudSkills", title: "SpringCloud", type: "danger" },
    { key: "MySQLSkills", title: "MySQL", type: "primary" },
    { key: "RedisSkills", title: "Redis", type: "success" },
    { key: "DevOpsSkills", title: "DevOps", type: "warning" },
    { key: "AiSkills", title: "AI与前沿技术", type: "danger" },
  ],
  JavaSkills: [
    "Java 8/11/17",
    "面向对象",
    "集合框架",
    "多线程",
    "JVM调优",
    "设计模式",
    "网络编程",
    "常用API",
  ],
  JavaWebSkills: [
    "HTML",
    "CSS",
    "JS",
    "VUE2/3",
    "ElementUI/ElementPlus",
    "Maven",
    "SpringBoot",
    "Mybatis/MybatisPlus",
    "SpringCache",
    "SpringTask",
    "Hibernate-validation",
    "Swagger/Knife4j",
  ],
  SpringCloudSkills: [
    "GateWay",
    "Eureka/Nacos",
    "XXL-Job",
    "Elastic Search",
    "OpenFeign/Dubbo",
    "Ribbon/LoadBalancer",
    "Hytrix/Sentinel",
    "RabbitMQ/Kafka/EMQ/RocketMQ",
    "Seata",
  ],
  MySQLSkills: [
    "SQL",
    "事务",
    "存储引擎",
    "索引",
    "SQL优化",
    "锁",
    "日志",
    "主从复制",
    "分库分表",
    "读写分离",
  ],
  RedisSkills: [
    "缓存",
    "会话管理",
    "消息队列",
    "分布式锁",
    "数据同步",
    "集群部署",
    "哨兵模式",
    "Replication",
  ],
  DevOpsSkills: ["Docker", "K8s", "Jenkins", "Git", "Linux", "AWS"],
  AiSkills: [
    "AI常用辅助工具(Calicat + Qoder + Devbox + Sealos)",
    "SpringAI",
    "Langchain",
    "Prompt",
    "Function Calling",
    "RAG",
    "Fine-tuning",
  ],
  skillProgress: [
    { name: "Java后端开发", percent: 98, color: "#67C23A" },
    { name: "分布式系统设计", percent: 92, color: "#67C23A" },
    { name: "AI Agent开发", percent: 90, color: "#67C23A" },
    { name: "系统架构设计", percent: 88, color: "#67C23A" },
    { name: "全栈开发", percent: 85, color: "#67C23A" },
  ],
  education: [
    {
      school: "上海交通大学",
      degree: "硕士",
      major: "计算机科学与技术",
      time: "2016.09 - 2019.06",
      gpa: "3.85/4.0",
      rank: "专业排名：前3%",
      description:
        "研究方向：分布式系统、人工智能\n发表EI论文2篇，获得软件著作权3项",
    },
    {
      school: "浙江大学",
      degree: "本科",
      major: "软件工程",
      time: "2012.09 - 2016.06",
      gpa: "3.9/4.0",
      rank: "校级优秀毕业生",
      description: "ACM国际大学生程序设计竞赛亚洲区银奖",
    },
  ],
  certificates: [
    {
      name: "Oracle Certified Professional, Java SE 17",
      time: "2023年 | Oracle认证",
    },
    {
      name: "AWS Certified Solutions Architect - Professional",
      time: "2022年 | 亚马逊AWS认证",
    },
    { name: "阿里云高级工程师认证（云计算方向）", time: "2021年 | 阿里云认证" },
    {
      name: "公司年度技术创新一等奖",
      time: "2023年 | 主导AI Agent平台研发获奖",
    },
    {
      name: "开源项目贡献者",
      time: "Spring、LangChain等知名开源项目Contributor",
    },
  ],
  workExperience: [
    {
      company: "字节跳动",
      position: "高级Java开发工程师",
      department: "抖音电商 | 技术部",
      time: "2021.04 - 至今",
      achievements: [
        "主导抖音电商订单中心系统重构，采用微服务架构设计，系统QPS从1万提升到10万，响应时间从200ms降低到50ms，可用性达到99.99%",
        "负责公司级AI Agent平台的架构设计与核心开发，基于LangChain和大模型技术，搭建了具备自主思考、工具调用、多Agent协作能力的智能代理平台",
        "带领5人技术团队，负责平台的全栈开发，后端采用Spring Boot + Cloud微服务架构，前端使用React，向量数据库采用Milvus，上线后累计接入业务场景20+，提升运营效率300%",
        "设计并实现分布式缓存架构、消息队列削峰填谷方案、数据库分库分表策略，支撑618、双11等大促活动平稳运行，零故障",
        "主导技术栈升级，推动Java 8到Java 17的迁移，Spring Cloud生态优化，研发效能提升40%，系统资源消耗降低25%",
      ],
      techStack: [
        "Java 17",
        "Spring Boot 3",
        "Spring Cloud",
        "AI Agent",
        "LangChain",
        "微服务架构",
        "分布式系统",
      ],
    },
    {
      company: "阿里巴巴集团",
      position: "Java开发工程师",
      department: "阿里云 | 中间件团队",
      time: "2019.07 - 2021.03",
      achievements: [
        "参与阿里云分布式消息队列RocketMQ的核心开发，负责消息存储引擎优化，消息写入性能提升30%，存储成本降低25%",
        "负责微服务治理平台的设计与开发，实现服务注册发现、配置中心、限流降级、链路追踪等核心功能，支撑公司上万服务实例运行",
        "设计并实现统一数据访问层框架，封装数据库操作、缓存、分库分表逻辑，减少重复代码开发，研发效率提升50%",
        "参与双十一全链路压测项目，负责中间件层面的性能优化和稳定性保障，帮助业务系统平稳支撑亿级流量冲击",
      ],
      techStack: [
        "Java 8/11",
        "Spring Boot",
        "Netty",
        "RocketMQ",
        "微服务治理",
        "分布式中间件",
      ],
    },
  ],
  projects: [
    {
      name: "企业级AI Agent开发平台",
      tags: [
        { name: "核心项目", type: "warning" },
        { name: "全栈开发", type: "success" },
      ],
      time: "2023.02 - 2023.10",
      role: "技术负责人",
      description:
        "打造低代码AI Agent开发平台，支持可视化搭建智能代理，集成多种大模型、工具调用、知识库、工作流编排等能力，帮助业务人员快速构建AI应用，无需编写代码。",
      techStack: {
        后端: "Java 17 + Spring Boot 3",
        AI引擎: "LangChain + AutoGPT",
        向量库: "Milvus + Elasticsearch",
        前端: "React + TypeScript",
        部署: "K8s + Docker",
      },
      results: [
        "平台上线后，累计创建AI Agent 5000+，覆盖客服、运营、研发等20+业务场景",
        "业务流程自动化率提升60%，节省人力成本超千万，获得公司年度技术创新一等奖",
        "Agent任务完成准确率达到92%，平均响应时间3s，支持同时处理上万并发请求",
      ],
    },
    {
      name: "抖音电商订单中心重构",
      tags: [
        { name: "核心项目", type: "warning" },
        { name: "后端开发", type: "primary" },
      ],
      time: "2021.06 - 2022.03",
      role: "核心开发工程师",
      description:
        "对原有订单系统进行微服务架构重构，解决单体架构性能瓶颈问题，支撑电商业务快速发展，满足亿级订单量处理需求。",
      techStack: {
        微服务架构: "Spring Cloud Alibaba",
        数据库: "MySQL + ShardingSphere分库分表",
        缓存: "Redis Cluster + Caffeine本地缓存",
        消息队列: "RocketMQ",
      },
      results: [
        "系统峰值QPS从1万提升到10万，平均响应时间从200ms降低到50ms，可用性达99.99%",
        "支撑双11当天订单创建量5亿+，系统平稳运行零故障",
        "系统扩展性大幅提升，新功能迭代周期从2周缩短到3天，研发效率提升70%",
      ],
    },
    {
      name: "开源AI Agent框架 SmartAgent",
      tags: [
        { name: "开源项目", type: "danger" },
        { name: "作者", type: "info" },
      ],
      time: "2022.10 - 至今",
      role: "核心开发者",
      description:
        "开源Java版AI Agent开发框架，提供简洁的API接口，帮助Java开发者快速构建AI代理应用，支持多模态、工具调用、记忆管理、多Agent协作等特性。",
      results: [
        "GitHub星标4.2k+，成为Java生态最受欢迎的AI Agent开发框架",
        "累计被1000+企业项目使用，包括多家知名互联网公司",
        "获得Gitee最有价值开源项目(GVP)称号，入选开源中国年度开源项目榜单",
      ],
    },
    {
      name: "微服务治理平台",
      tags: [
        { name: "平台项目", type: "primary" },
        { name: "核心开发", type: "success" },
      ],
      time: "2020.03 - 2020.12",
      role: "核心开发工程师",
      description:
        "打造一站式微服务治理平台，集成服务注册发现、配置中心、限流降级、熔断保护、链路追踪、监控告警等能力，为公司微服务架构提供统一治理解决方案。",
      results: [
        "平台接入公司所有微服务，管理服务实例10万+，覆盖全部业务线",
        "微服务故障发生率降低70%，故障定位时间从平均1小时缩短到5分钟",
        "服务发布效率提升300%，支持灰度发布、金丝雀发布等多种发布策略",
      ],
    },
  ],
});

// 计算属性：个人简介段落
const introParagraphs = computed(() => {
  return resumeData.introduction.split("\n").filter((p) => p.trim());
});

// 方法：添加技能
const addSkill = (category, skill) => {
  if (skill && skill.trim()) {
    resumeData[category].push(skill.trim());
  }
};

// 方法：从输入框添加技能
const addSkillFromInput = (category) => {
  const value = skillInputValues[category]?.trim();
  if (value) {
    resumeData[category].push(value);
    skillInputValues[category] = "";
  }
};

// 方法：删除技能
const removeSkill = (category, index) => {
  resumeData[category].splice(index, 1);
};

// 方法：添加技能分类
const tagTypes = [
  "success",
  "warning",
  "danger",
  "primary",
  "success",
  "warning",
  "danger",
  "primary",
  "success",
  "warning",
];
const addSkillCategory = () => {
  const newKey = "customSkills" + Date.now();
  const typeIndex = resumeData.skillCategories.length % tagTypes.length;
  resumeData.skillCategories.push({
    key: newKey,
    title: "新分类",
    type: tagTypes[typeIndex],
  });
  resumeData[newKey] = [];
};

// 方法：删除技能分类
const removeSkillCategory = (index) => {
  const category = resumeData.skillCategories[index];
  if (category) {
    resumeData.skillCategories.splice(index, 1);
    delete resumeData[category.key];
  }
};

// 方法：处理头像上传
const handleAvatarChange = (file) => {
  const reader = new FileReader();
  reader.onload = (e) => {
    resumeData.basic.avatar = e.target.result;
  };
  reader.readAsDataURL(file.raw);
};

// 方法：导出PDF
const exportPDF = async () => {
  if (!cvContent.value) return;

  isExporting.value = true;

  try {
    // 先切换到预览模式
    const wasEditMode = isEditMode.value;
    isEditMode.value = false;

    // 临时隐藏编辑面板
    const editPanel = document.querySelector(".edit-panel");
    const editToggle = document.querySelector(".edit-toggle");
    if (editPanel) editPanel.style.display = "none";
    if (editToggle) editToggle.style.display = "none";

    // 等待DOM更新和头像加载
    await new Promise((resolve) => setTimeout(resolve, 1500));

    const element = cvContent.value;

    // 获取实际内容高度
    const contentHeight = element.scrollHeight;
    const contentWidth = element.scrollWidth;

    // 预加载头像图片
    const avatarImg = new Image();
    avatarImg.crossOrigin = "anonymous";
    await new Promise((resolve) => {
      avatarImg.onload = resolve;
      avatarImg.onerror = resolve;
      avatarImg.src = resumeData.basic.avatar;
    });

    const canvas = await html2canvas(element, {
      scale: 2,
      useCORS: true,
      allowTaint: true,
      logging: false,
      backgroundColor: "#f5f7fa",
      height: contentHeight,
      width: contentWidth,
      windowHeight: contentHeight,
      windowWidth: contentWidth,
      imageTimeout: 0,
      foreignObjectRendering: false,
    });

    const imgData = canvas.toDataURL("image/png");
    const pdf = new jsPDF("p", "mm", "a4");

    const pdfWidth = pdf.internal.pageSize.getWidth();
    const pdfHeight = pdf.internal.pageSize.getHeight();

    // 计算缩放比例，让内容填满页面宽度
    const imgWidth = canvas.width;
    const imgHeight = canvas.height;
    const scaleFactor = pdfWidth / imgWidth;
    const scaledHeight = imgHeight * scaleFactor;

    let heightLeft = scaledHeight;
    let position = 0;
    let pageCount = 0;

    // 添加图片到PDF，处理多页
    while (heightLeft > 0) {
      if (pageCount > 0) {
        pdf.addPage();
      }

      // 计算当前页要显示的图片部分
      const sourceY = (pageCount * pdfHeight) / scaleFactor;
      const sourceHeight = Math.min(
        pdfHeight / scaleFactor,
        imgHeight - sourceY
      );
      const destHeight = sourceHeight * scaleFactor;

      pdf.addImage(
        imgData,
        "PNG",
        0,
        -position,
        pdfWidth,
        scaledHeight,
        undefined,
        "FAST"
      );

      heightLeft -= pdfHeight;
      position += pdfHeight;
      pageCount++;

      // 限制最大页数防止无限循环
      if (pageCount > 10) break;
    }

    pdf.save(`${resumeData.basic.name || "个人简历"}.pdf`);

    // 恢复之前的模式
    isEditMode.value = wasEditMode;

    // 恢复编辑面板显示
    if (editPanel) editPanel.style.display = "";
    if (editToggle) editToggle.style.display = "";

    ElMessage.success("PDF导出成功！");
  } catch (error) {
    console.error("PDF导出失败:", error);
    ElMessage.error("PDF导出失败，请重试");
  } finally {
    isExporting.value = false;
  }
};

// 方法：拖拽排序
const handleDragStart = (index) => {
  dragIndex.value = index;
};

const handleDragEnter = (index) => {
  dragOverIndex.value = index;
};

const handleDrop = (index) => {
  if (dragIndex.value !== -1 && dragIndex.value !== index) {
    const item = resumeData.skillCategories[dragIndex.value];
    resumeData.skillCategories.splice(dragIndex.value, 1);
    resumeData.skillCategories.splice(index, 0, item);
  }
  dragIndex.value = -1;
  dragOverIndex.value = -1;
};

// 方法：获取标签类型（处理新增分类的info类型）
const getTagType = (type) => {
  const validTypes = ["success", "warning", "danger", "primary"];
  return validTypes.includes(type) ? type : "primary";
};

// 方法：教育经历
const addEducation = () => {
  resumeData.education.push({
    school: "",
    degree: "",
    major: "",
    time: "",
    gpa: "",
    rank: "",
    description: "",
  });
};

const removeEducation = (index) => {
  resumeData.education.splice(index, 1);
};

// 方法：证书
const addCertificate = () => {
  resumeData.certificates.push({
    name: "",
    time: "",
  });
};

const removeCertificate = (index) => {
  resumeData.certificates.splice(index, 1);
};

// 方法：工作经历
const addWork = () => {
  resumeData.workExperience.push({
    company: "",
    position: "",
    department: "",
    time: "",
    achievements: [],
    techStack: [],
  });
};

const removeWork = (index) => {
  resumeData.workExperience.splice(index, 1);
};

const addAchievement = (workIndex) => {
  resumeData.workExperience[workIndex].achievements.push("");
};

const removeAchievement = (workIndex, achievementIndex) => {
  resumeData.workExperience[workIndex].achievements.splice(achievementIndex, 1);
};

const addWorkTech = (workIndex, event) => {
  const tech = event.target.value.trim();
  if (tech) {
    resumeData.workExperience[workIndex].techStack.push(tech);
    event.target.value = "";
  }
};

// 方法：项目经验
const addProject = () => {
  resumeData.projects.push({
    name: "",
    tags: [],
    time: "",
    role: "",
    description: "",
    results: [],
  });
};

const removeProject = (index) => {
  resumeData.projects.splice(index, 1);
};

const addProjectResult = (projectIndex) => {
  resumeData.projects[projectIndex].results.push("");
};

const removeProjectResult = (projectIndex, resultIndex) => {
  resumeData.projects[projectIndex].results.splice(resultIndex, 1);
};

// 方法：添加项目标签
const addProjectTag = (projectIndex) => {
  const tagName = projectTagInput[projectIndex]?.trim();
  if (tagName) {
    const tagTypes = ["success", "warning", "danger", "primary", "info"];
    const typeIndex =
      resumeData.projects[projectIndex].tags.length % tagTypes.length;
    resumeData.projects[projectIndex].tags.push({
      name: tagName,
      type: tagTypes[typeIndex],
    });
    projectTagInput[projectIndex] = "";
  }
};

// 方法：添加项目技术架构
const addProjectTechStack = (projectIndex) => {
  const key = newTechKey[projectIndex]?.trim();
  const value = newTechValue[projectIndex]?.trim();
  if (key && value) {
    if (!resumeData.projects[projectIndex].techStack) {
      resumeData.projects[projectIndex].techStack = {};
    }
    resumeData.projects[projectIndex].techStack[key] = value;
    newTechKey[projectIndex] = "";
    newTechValue[projectIndex] = "";
  }
};

// 方法：工作经历拖拽排序
const handleWorkDragStart = (index) => {
  workDragIndex.value = index;
};

const handleWorkDragOver = (index) => {
  // 计算应该插入的位置
  const rect = event.currentTarget.getBoundingClientRect();
  const midY = rect.top + rect.height / 2;
  const mouseY = event.clientY;

  // 如果在元素上半部分，插入到当前位置；如果在下半部分，插入到下一个位置
  if (mouseY < midY) {
    workDragOverIndex.value = index;
  } else {
    workDragOverIndex.value = index + 1;
  }
};

const handleWorkDragEnter = (index) => {
  // 阻止默认行为，允许放置
  event.preventDefault();
};

const handleWorkDrop = (index) => {
  if (workDragIndex.value !== -1) {
    const item = resumeData.workExperience[workDragIndex.value];
    resumeData.workExperience.splice(workDragIndex.value, 1);

    // 根据拖拽目标位置确定插入位置
    let insertIndex = workDragOverIndex.value;
    if (insertIndex > workDragIndex.value) {
      insertIndex--;
    }

    // 确保插入位置在有效范围内
    insertIndex = Math.max(
      0,
      Math.min(insertIndex, resumeData.workExperience.length)
    );
    resumeData.workExperience.splice(insertIndex, 0, item);
  }
  workDragIndex.value = -1;
  workDragOverIndex.value = -1;
};

const handleWorkDragEnd = () => {
  workDragIndex.value = -1;
  workDragOverIndex.value = -1;
};

// 方法：项目经验拖拽排序
const handleProjectDragStart = (index) => {
  projectDragIndex.value = index;
};

const handleProjectDragOver = (index) => {
  // 计算应该插入的位置
  const rect = event.currentTarget.getBoundingClientRect();
  const midY = rect.top + rect.height / 2;
  const mouseY = event.clientY;

  // 如果在元素上半部分，插入到当前位置；如果在下半部分，插入到下一个位置
  if (mouseY < midY) {
    projectDragOverIndex.value = index;
  } else {
    projectDragOverIndex.value = index + 1;
  }
};

const handleProjectDragEnter = (index) => {
  // 阻止默认行为，允许放置
  event.preventDefault();
};

const handleProjectDrop = (index) => {
  if (projectDragIndex.value !== -1) {
    const item = resumeData.projects[projectDragIndex.value];
    resumeData.projects.splice(projectDragIndex.value, 1);

    // 根据拖拽目标位置确定插入位置
    let insertIndex = projectDragOverIndex.value;
    if (insertIndex > projectDragIndex.value) {
      insertIndex--;
    }

    // 确保插入位置在有效范围内
    insertIndex = Math.max(
      0,
      Math.min(insertIndex, resumeData.projects.length)
    );
    resumeData.projects.splice(insertIndex, 0, item);
  }
  projectDragIndex.value = -1;
  projectDragOverIndex.value = -1;
};

const handleProjectDragEnd = () => {
  projectDragIndex.value = -1;
  projectDragOverIndex.value = -1;
};

// 方法：证书荣誉拖拽排序
const handleCertDragStart = (index) => {
  certDragIndex.value = index;
};

const handleCertDragOver = (index) => {
  const rect = event.currentTarget.getBoundingClientRect();
  const midY = rect.top + rect.height / 2;
  const mouseY = event.clientY;

  if (mouseY < midY) {
    certDragOverIndex.value = index;
  } else {
    certDragOverIndex.value = index + 1;
  }
};

const handleCertDragEnter = (index) => {
  event.preventDefault();
};

const handleCertDrop = (index) => {
  if (certDragIndex.value !== -1) {
    const item = resumeData.certificates[certDragIndex.value];
    resumeData.certificates.splice(certDragIndex.value, 1);

    let insertIndex = certDragOverIndex.value;
    if (insertIndex > certDragIndex.value) {
      insertIndex--;
    }

    insertIndex = Math.max(
      0,
      Math.min(insertIndex, resumeData.certificates.length)
    );
    resumeData.certificates.splice(insertIndex, 0, item);
  }
  certDragIndex.value = -1;
  certDragOverIndex.value = -1;
};

const handleCertDragEnd = () => {
  certDragIndex.value = -1;
  certDragOverIndex.value = -1;
};

// 方法：教育背景拖拽排序
const handleEduDragStart = (index) => {
  eduDragIndex.value = index;
};

const handleEduDragOver = (index) => {
  const rect = event.currentTarget.getBoundingClientRect();
  const midY = rect.top + rect.height / 2;
  const mouseY = event.clientY;

  if (mouseY < midY) {
    eduDragOverIndex.value = index;
  } else {
    eduDragOverIndex.value = index + 1;
  }
};

const handleEduDragEnter = (index) => {
  event.preventDefault();
};

const handleEduDrop = (index) => {
  if (eduDragIndex.value !== -1) {
    const item = resumeData.education[eduDragIndex.value];
    resumeData.education.splice(eduDragIndex.value, 1);

    let insertIndex = eduDragOverIndex.value;
    if (insertIndex > eduDragIndex.value) {
      insertIndex--;
    }

    insertIndex = Math.max(
      0,
      Math.min(insertIndex, resumeData.education.length)
    );
    resumeData.education.splice(insertIndex, 0, item);
  }
  eduDragIndex.value = -1;
  eduDragOverIndex.value = -1;
};

const handleEduDragEnd = () => {
  eduDragIndex.value = -1;
  eduDragOverIndex.value = -1;
};

// 方法：技术栈拖拽排序（增强版）
const handleSkillDragOver = (index) => {
  const rect = event.currentTarget.getBoundingClientRect();
  const midY = rect.top + rect.height / 2;
  const mouseY = event.clientY;

  if (mouseY < midY) {
    dragOverIndex.value = index;
  } else {
    dragOverIndex.value = index + 1;
  }
};

const handleSkillDragEnter = (index) => {
  event.preventDefault();
};

const handleDragEnd = () => {
  dragIndex.value = -1;
  dragOverIndex.value = -1;
};
</script>

<style scoped>
.cv-container {
  max-width: 1200px;
  margin: 0 auto;
  background: #f5f7fa;
  min-height: 100vh;
}

/* 顶部区域 */
.header-section {
  background: linear-gradient(135deg, #10b981 0%, #059669 100%);
  padding: 30px 40px;
  color: white;
}

.header-content {
  display: flex;
  align-items: center;
  gap: 30px;
}

.avatar-section {
  flex-shrink: 0;
}

.avatar {
  border: 4px solid rgba(255, 255, 255, 0.3);
  border-radius: 50%;
  padding: 4px;
}

.info-section {
  flex: 1;
}

.name-row {
  display: flex;
  align-items: center;
  gap: 15px;
  margin-bottom: 10px;
}

.name {
  font-size: 32px;
  font-weight: bold;
  margin: 0;
}

.title-tag {
  font-size: 14px;
}

.subtitle {
  font-size: 16px;
  opacity: 0.9;
  margin-bottom: 15px;
}

.contact-row {
  display: flex;
  flex-wrap: wrap;
  gap: 20px;
}

.contact-item {
  display: flex;
  align-items: center;
  gap: 5px;
  font-size: 14px;
  opacity: 0.9;
}

/* 主体内容 */
.main-content {
  display: flex;
  gap: 20px;
  padding: 20px;
}

.left-sidebar {
  width: 320px;
  flex-shrink: 0;
}

.right-content {
  flex: 1;
}

/* 通用区块样式 */
.section {
  background: white;
  border-radius: 8px;
  padding: 20px;
  margin-bottom: 20px;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.05);
}

.section-title {
  font-size: 18px;
  font-weight: bold;
  color: #333;
  margin-bottom: 20px;
  display: flex;
  align-items: center;
  gap: 8px;
  padding-bottom: 10px;
  border-bottom: 2px solid #e4e7ed;
}

.section-title .el-icon {
  color: #10b981;
}

/* 技术栈 */
.skill-category {
  margin-bottom: 15px;
}

.skill-category h3 {
  font-size: 14px;
  color: #666;
  margin-bottom: 10px;
}

.skill-tags {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}

.skill-tag {
  margin: 0;
}

.skill-progress {
  margin-top: 20px;
}

.progress-item {
  margin-bottom: 12px;
}

.progress-label {
  display: flex;
  justify-content: space-between;
  font-size: 13px;
  color: #666;
  margin-bottom: 5px;
}

/* 教育背景 */
.education-item {
  margin-bottom: 20px;
  padding-bottom: 15px;
  border-bottom: 1px solid #ebeef5;
}

.education-item:last-child {
  border-bottom: none;
  margin-bottom: 0;
  padding-bottom: 0;
}

.edu-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 8px;
}

.school-name {
  font-size: 16px;
  font-weight: bold;
  color: #333;
  margin: 0;
}

.edu-tag {
  background: #10b981;
  color: white;
  padding: 2px 8px;
  border-radius: 4px;
  font-size: 12px;
}

.edu-major {
  font-size: 14px;
  color: #666;
  margin-bottom: 5px;
}

.edu-time {
  font-size: 13px;
  color: #999;
  margin-bottom: 5px;
}

.edu-detail {
  font-size: 13px;
  color: #666;
  margin-bottom: 5px;
}

.edu-desc {
  font-size: 12px;
  color: #999;
  line-height: 1.6;
  white-space: pre-line;
}

/* 证书 */
.cert-list {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.cert-item {
  display: flex;
  align-items: flex-start;
  gap: 10px;
}

.cert-icon {
  color: #f59e0b;
  flex-shrink: 0;
  margin-top: 2px;
}

.cert-name {
  font-size: 14px;
  color: #333;
  margin-bottom: 3px;
}

.cert-time {
  font-size: 12px;
  color: #999;
}

/* 个人简介 */
.intro-content p {
  font-size: 14px;
  line-height: 1.8;
  color: #666;
  margin-bottom: 10px;
}

.intro-content p:last-child {
  margin-bottom: 0;
}

/* 工作经历 */
.work-item {
  margin-bottom: 25px;
  padding-bottom: 20px;
  border-bottom: 1px solid #ebeef5;
}

.work-item:last-child {
  border-bottom: none;
  margin-bottom: 0;
  padding-bottom: 0;
}

.work-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 5px;
}

.company-info {
  display: flex;
  align-items: center;
  gap: 10px;
}

.company-name {
  font-size: 18px;
  font-weight: bold;
  color: #333;
  margin: 0;
}

.work-position {
  font-size: 12px;
}

.work-time {
  font-size: 14px;
  color: #10b981;
  font-weight: 500;
}

.work-dept {
  font-size: 13px;
  color: #999;
  margin-bottom: 10px;
}

.work-content h4 {
  font-size: 14px;
  color: #333;
  margin: 10px 0 8px 0;
}

.work-content ul {
  margin: 0;
  padding-left: 18px;
}

.work-content li {
  font-size: 13px;
  color: #666;
  line-height: 1.8;
  margin-bottom: 5px;
}

.tech-tags {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  margin-top: 8px;
}

/* 项目经验 */
.project-item {
  margin-bottom: 25px;
  padding-bottom: 20px;
  border-bottom: 1px solid #ebeef5;
}

.project-item:last-child {
  border-bottom: none;
  margin-bottom: 0;
  padding-bottom: 0;
}

.project-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 8px;
}

.project-name {
  font-size: 16px;
  font-weight: bold;
  color: #333;
  margin: 0;
}

.project-tags {
  display: flex;
  gap: 8px;
}

.project-info {
  font-size: 13px;
  color: #999;
  margin-bottom: 10px;
}

.project-desc {
  font-size: 13px;
  color: #666;
  line-height: 1.8;
  margin-bottom: 10px;
}

.project-tech h4,
.project-result h4 {
  font-size: 13px;
  color: #333;
  margin: 10px 0 8px 0;
}

.tech-arch {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
}

.tech-item {
  display: flex;
  align-items: center;
  gap: 5px;
}

.tech-label {
  font-size: 12px;
  color: #666;
}

.project-result ul {
  margin: 0;
  padding-left: 18px;
}

.project-result li {
  font-size: 13px;
  color: #666;
  line-height: 1.8;
  margin-bottom: 5px;
}

/* 编辑面板样式 */
.edit-toggle {
  position: fixed;
  top: 20px;
  right: 20px;
  z-index: 1000;
}

/* PDF导出时隐藏的元素 */
.no-print {
  display: block;
}

/* 简历内容区域 */
.cv-content {
  width: 100%;
  position: relative;
}

/* 分页线样式 */
.page-break-lines {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  pointer-events: none;
  z-index: 100;
}

.page-break-line {
  position: absolute;
  left: 0;
  right: 0;
  height: 2px;
  background: repeating-linear-gradient(
    90deg,
    #ff6b6b,
    #ff6b6b 10px,
    transparent 10px,
    transparent 20px
  );
  border-top: 1px dashed #ff6b6b;
}

.page-break-label {
  position: absolute;
  right: 20px;
  top: -20px;
  background: #ff6b6b;
  color: white;
  padding: 2px 8px;
  border-radius: 4px;
  font-size: 12px;
  font-weight: bold;
}

.edit-panel {
  background: white;
  padding: 20px;
  margin: 20px;
  border-radius: 8px;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.1);
}

.edit-section {
  margin-bottom: 20px;
  padding: 15px;
  background: #f5f7fa;
  border-radius: 6px;
  cursor: move;
  transition: all 0.3s;
}

.edit-section.dragging {
  opacity: 0.5;
  border: 2px dashed #409eff;
}

.edit-section h4 {
  margin: 0 0 10px 0;
  color: #333;
  display: flex;
  align-items: center;
  gap: 8px;
}

.drag-handle {
  cursor: move;
  color: #909399;
}

.section-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 15px;
}

.tag-editor {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  align-items: center;
}

.edit-tag {
  margin: 0;
}

.skill-edit-item {
  display: flex;
  align-items: center;
  margin-bottom: 10px;
}

.list-item {
  display: flex;
  gap: 10px;
  margin-bottom: 10px;
  align-items: flex-start;
}

/* 工作业绩编辑项 */
.achievement-item {
  display: flex;
  gap: 10px;
  margin-bottom: 10px;
  align-items: flex-start;
  width: 100%;
}

.achievement-item .el-input {
  flex: 1;
}

/* 项目成果编辑项 */
.result-item {
  display: flex;
  gap: 10px;
  margin-bottom: 10px;
  align-items: flex-start;
  width: 100%;
}

.result-item .el-input {
  flex: 1;
}

/* 技术架构编辑项 */
.tech-stack-container {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 12px;
}

.tech-stack-item {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 10px 12px;
  background-color: #f5f7fa;
  border-radius: 6px;
}

.tech-key {
  font-weight: 500;
  color: #606266;
  min-width: 70px;
  flex-shrink: 0;
  white-space: nowrap;
}

.tech-stack-item .el-input {
  flex: 1;
  min-width: 0;
}

.add-tech-stack {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  padding: 8px 12px;
  background-color: #f0f9ff;
  border: 1px dashed #409eff;
  border-radius: 6px;
  width: auto;
  justify-self: start;
}

.tech-separator {
  color: #909399;
  font-weight: 500;
}

/* 头像编辑 */
.avatar-edit {
  display: flex;
  align-items: center;
  gap: 10px;
}

.avatar-uploader {
  display: inline-block;
}

/* 拖拽排序样式 */
.drag-handle {
  display: flex;
  align-items: center;
  gap: 8px;
  cursor: move;
}

.drag-handle .el-icon {
  color: #909399;
  font-size: 16px;
}

.drag-handle:hover .el-icon {
  color: #409eff;
}

/* 拖拽插入指示器样式 */
.drop-indicator {
  position: relative;
  margin: 8px 0;
  height: 2px;
}

.drop-indicator-top {
  margin-top: -10px;
  margin-bottom: 8px;
}

.drop-indicator-bottom {
  margin-top: 8px;
  margin-bottom: -10px;
}

.drop-indicator-line {
  position: absolute;
  left: 0;
  right: 0;
  height: 3px;
  background: linear-gradient(90deg, #409eff, #67c23a);
  border-radius: 2px;
  animation: pulse 1s ease-in-out infinite;
}

.drop-indicator-label {
  position: absolute;
  left: 50%;
  top: -22px;
  transform: translateX(-50%);
  background: #409eff;
  color: white;
  padding: 2px 8px;
  border-radius: 4px;
  font-size: 12px;
  white-space: nowrap;
  z-index: 10;
}

@keyframes pulse {
  0%,
  100% {
    opacity: 1;
  }
  50% {
    opacity: 0.6;
  }
}

/* 拖拽时的目标元素样式 */
.edit-section.drag-over {
  border: 2px dashed #409eff;
  background: #f0f9ff;
}

.edit-section {
  transition: all 0.3s ease;
}

.edit-section.drag-over {
  border: 2px dashed #409eff;
  background: #ecf5ff;
}

/* 响应式 */
@media (max-width: 900px) {
  .main-content {
    flex-direction: column;
  }

  .left-sidebar {
    width: 100%;
  }

  .header-content {
    flex-direction: column;
    text-align: center;
  }

  .contact-row {
    justify-content: center;
  }

  .name-row {
    flex-wrap: wrap;
    justify-content: center;
  }

  .edit-toggle {
    position: relative;
    top: 0;
    right: 0;
    margin: 10px;
    text-align: center;
  }
}
</style>

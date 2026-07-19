<template>
  <section class="glass-card showcase-container">
    <div class="showcase-header">
      <div class="showcase-title-group">
        <h2 class="showcase-title">宝妈成长与生活平衡案例库 (30 精选样例)</h2>
        <p class="showcase-subtitle">体验不同阶段的副业探索变现、碎片时间利用、产后减脂塑形与自我赋能，点击“一键同款生成”即可即刻核算</p>
      </div>
      <div class="showcase-badge">自我赋能 · 免费体验</div>
    </div>

    <!-- 搜索与分类筛选 -->
    <div class="showcase-filter-bar">
      <div class="category-tabs">
        <button 
          v-for="cat in categories" 
          :key="cat"
          class="category-tab"
          :class="{ active: currentCategory === cat }"
          @click="currentCategory = cat"
        >
          {{ cat }}
        </button>
      </div>
      <div class="search-input-wrapper">
        <input 
          v-model="searchQuery"
          type="text"
          placeholder="搜索宝妈阶段、副业方向、时间管理、悦己塑形、流派或关键字..."
          class="search-input"
        />
      </div>
    </div>

    <!-- 样例列表格 Grid -->
    <div class="sample-grid">
      <div 
        v-for="sample in paginatedSamples" 
        :key="sample.id" 
        class="sample-card"
      >
        <div class="sample-card-header">
          <span class="topic-category-tag">{{ sample.category }}</span>
          <span class="style-name-tag">{{ sample.style }}</span>
        </div>
        <div class="sample-original">
          <span class="sample-label">宝妈探索：</span>“{{ sample.destination }}，诉求：{{ sample.topic }}”
        </div>
        <div class="sample-rewritten">
          <span class="sample-label">赋能方案：</span>{{ sample.core }}
        </div>
        <div class="sample-card-footer">
          <button class="use-sample-btn" @click="$emit('use-sample', sample.topic, sample.destination)">
            一键同款生成
          </button>
        </div>
      </div>
    </div>

    <!-- 空状态提示 -->
    <div v-if="filteredSamples.length === 0" class="empty-showcase">
      未找到匹配的宝妈成长案例，请尝试切换分类或重置搜索关键词。
    </div>

    <!-- 分页组件 -->
    <div v-if="filteredSamples.length > pageSize" class="pagination-bar">
      <button 
        class="page-btn" 
        :disabled="currentPage === 1"
        @click="currentPage--"
      >
        上一页
      </button>
      <span class="page-info">第 {{ currentPage }} / {{ totalPages }} 页 (共 {{ filteredSamples.length }} 条)</span>
      <button 
        class="page-btn" 
        :disabled="currentPage === totalPages"
        @click="currentPage++"
      >
        下一页
      </button>
    </div>
  </section>
</template>

<script setup lang="ts">
import { ref, computed, watch } from 'vue';

defineEmits<{
  (e: 'use-sample', text: string, destination: string): void;
}>();

const categories = ['全部', '副业探索', '时间管理', '产后悦己', '职场复工', '家庭和谐'];
const currentCategory = ref('全部');
const searchQuery = ref('');
const currentPage = ref(1);
const pageSize = 6;

interface BaomaSample {
  id: number;
  category: string;
  destination: string;
  topic: string;
  style: string;
  core: string;
}

// 精选 30 条宝妈成长案例
const raw30Samples: BaomaSample[] = [
  {
    id: 1,
    category: '副业探索',
    destination: '全职宝妈 (宝宝1岁半)',
    topic: '求小红书家居育儿博主定位与居家文案写作，寻找月入三千副业。',
    style: '宝妈副业与变现探索流',
    core: '方案：聚焦“小户型高效收纳+科学辅食打卡”垂直赛道，每日利用宝宝小睡1小时创作图文，对接蒲公英广告变现。'
  },
  {
    id: 2,
    category: '时间管理',
    destination: '二胎宝妈 (大宝4岁，小宝8个月)',
    topic: '每天照顾两个孩子精疲力竭，时间被切割严重，求家务与作息安排。',
    style: '时间管理与高效带娃流',
    core: '方案：引入“家务批处理法则”（晨间15分钟爆破清洁+预制菜备餐），建立双宝重叠午睡2小时为妈妈专属充电时间。'
  },
  {
    id: 3,
    category: '产后悦己',
    destination: '产后半年宝妈',
    topic: '身材走样肚子松弛，腹直肌分离2指，求在家轻运动与心态调整。',
    style: '身心塑形与产后悦己流',
    core: '方案：每日15分钟普拉提腹脑呼吸与腹直肌安全训练，搭配低 GI 高蛋白膳食，谢绝体貌焦虑，重塑造自信。'
  },
  {
    id: 4,
    category: '职场复工',
    destination: '产后4个月复工宝妈',
    topic: '面临职场边缘化危机与背奶压力，求建立工作边界与效率提升。',
    style: '职场复工与平衡破局流',
    core: '方案：制定背奶清爽时间表，建立“完成大于完美”职场边界，主动向主管汇报关键 KPI 数据，重塑专业度。'
  },
  {
    id: 5,
    category: '家庭和谐',
    destination: '全职宝妈 (宝宝2岁)',
    topic: '因家庭开支与育儿观念与老公发生争执，求财务自主与沟通。',
    style: '温馨家庭与亲密关系流',
    core: '方案：推行家庭“三账户”财务精细化理财（育儿基金+家庭开支+宝妈悦己金），用非暴力沟通表达情绪与诉求。'
  },
  {
    id: 6,
    category: '副业探索',
    destination: '全职宝妈 (宝宝3岁已入园)',
    topic: '空闲时间增多，想学习视频剪辑与电商带货。',
    style: '宝妈副业与变现探索流',
    core: '方案：拆解短视频剪辑脚本基础，建立母婴好物体验佣金带货，从无货源橱窗试水，月入零花钱。'
  },
  {
    id: 7,
    category: '时间管理',
    destination: '新手宝妈 (宝宝3个月)',
    topic: '零碎时间无法控制，睡眠严重不足，求睡眠补充。',
    style: '时间管理与高效带娃流',
    core: '方案：实行“随宝同睡”能量恢复法，授权队友分担夜间一次喂奶，开启 20 分钟高效冥想恢复法。'
  },
  {
    id: 8,
    category: '产后悦己',
    destination: '全职宝妈 (宝宝2岁半)',
    topic: '长期不买新衣服妆容邋遢，想重新找回优雅美丽。',
    style: '身心塑形与产后悦己流',
    core: '方案：每周设置一次“宝妈独立日”出去喝咖啡看展，建立基础胶囊衣橱与5分钟精致淡妆仪式。'
  },
  {
    id: 9,
    category: '职场复工',
    destination: '二胎复工宝妈',
    topic: '加急加班频繁，无法兼顾两个孩子作业，求家庭分工。',
    style: '职场复工与平衡破局流',
    core: '方案：召开家庭会议明确队友分工卡片（队友负责大宝作业，宝妈负责小宝睡眠），必要时采购洗碗机与扫地机。'
  },
  {
    id: 10,
    category: '家庭和谐',
    destination: '三代同堂育儿宝妈',
    topic: '婆婆育儿观念陈旧爱追饭喂饭，求情商化解与界限。',
    style: '温馨家庭与亲密关系流',
    core: '方案：借第三方儿科专家视频委婉表达，给予长辈充分尊重与情绪价值，核心底线坚守原则。'
  },
  {
    id: 11,
    category: '副业探索',
    destination: '自由职业宝妈 (宝宝1岁)',
    topic: '擅长手作烘焙，想发展同城团购与私域客户。',
    style: '宝妈副业与变现探索流',
    core: '方案：打造同城无添加婴儿无糖米饼与节日烘焙礼盒，利用朋友圈社群预售，建立忠实妈妈私域圈。'
  },
  {
    id: 12,
    category: '时间管理',
    destination: '职场兼带娃宝妈',
    topic: '下班回家做饭做家务如打仗，求一周预制备餐套路。',
    style: '时间管理与高效带娃流',
    core: '方案：周末统一进行“半成品冷冻备餐”（如切好肉片、包好饺子、洗好切蔬菜包），工作日15分钟快速出锅。'
  },
  {
    id: 13,
    category: '产后悦己',
    destination: '产后1年宝妈',
    topic: '情绪低落易怒，动不动想发火，求情绪宣泄与调养。',
    style: '身心塑形与产后悦己流',
    core: '方案：建立情绪“暂停按键”，发火前深呼吸5秒；练习书写情绪日记，补充B族维生素与镁元素安神。'
  },
  {
    id: 14,
    category: '职场复工',
    destination: '跳槽复工宝妈',
    topic: '求职被问“如何平衡工作与家庭”，求标准高情商回答。',
    style: '职场复工与平衡破局流',
    core: '方案：突出家庭支持系统完善（有可靠托育/长辈带娃），强调成熟宝妈的高效时间管理与专注执行力。'
  },
  {
    id: 15,
    category: '家庭和谐',
    destination: '全职宝妈 (宝宝8个月)',
    topic: '感觉老公下班玩手机不帮忙带娃，求沟通引诱。',
    style: '温馨亲子与正面管教流',
    core: '方案：给老公分配明确具体的亲子任务（如“负责给宝宝洗澡与讲睡前故事”），及时夸奖放大其成就感。'
  },
  {
    id: 16,
    category: '副业探索',
    destination: '文案特长宝妈',
    topic: '想从事公众号兼职撰稿与小红书代运营。',
    style: '宝妈副业与变现探索流',
    core: '方案：建立个人拆书与母婴品牌软文作品集，在远程招聘平台投递兼职，实现按件结账自由。'
  },
  {
    id: 17,
    category: '时间管理',
    destination: '全职宝妈 (宝宝10个月)',
    topic: '一天到晚收拾玩具依然一片狼藉，求玩具收纳系统。',
    style: '时间管理与高效带娃流',
    core: '方案：采用“玩具轮换机制”（仅保留4套玩具在客厅，其余收纳闭门），游戏结束带领宝宝一起游戏化归位。'
  },
  {
    id: 18,
    category: '产后悦己',
    destination: '产后2年宝妈',
    topic: '想考取营养师证书或心理咨询师，求学习规划。',
    style: '身心塑形与产后悦己流',
    core: '方案：拆解考证大纲为每日20分钟听课音轨（洗漱听课+带娃遛弯听课），利用微习惯3个月高效通关。'
  },
  {
    id: 19,
    category: '职场复工',
    destination: '管理层复工宝妈',
    topic: '出差频繁舍不得宝宝，求克服分离焦虑。',
    style: '职场复工与平衡破局流',
    core: '方案：出差期间定时视频高质量互动，准备“妈妈留下的神秘礼物盒”，高质量陪伴重于时间长度。'
  },
  {
    id: 20,
    category: '家庭和谐',
    destination: '全职宝妈 (宝宝1岁)',
    topic: '想买自己喜欢的东西心里有愧疚感，求建立财权。',
    style: '温馨家庭与亲密关系流',
    core: '方案：明确全职带娃的家庭经济贡献价值，在家庭预算中设立不可侵犯的“宝妈自由支配基金”。'
  },
  {
    id: 21,
    category: '副业探索',
    destination: '英语专业宝妈',
    topic: '想在家开展幼儿英语启蒙与同城绘本馆打卡。',
    style: '宝妈副业与变现探索流',
    core: '方案：记录自家宝宝英语启蒙成长轨迹，输出音标与绘本朗读短视频，吸引同城妈妈付费社群。'
  },
  {
    id: 22,
    category: '时间管理',
    destination: '二胎职场宝妈',
    topic: '早晨出门像打仗，经常迟到，求晨间高效流程。',
    style: '时间管理与高效带娃流',
    core: '方案：将所有衣物、书包、奶瓶备齐工作转移至“前一晚九点”完成，早晨实现流水线15分钟出门。'
  },
  {
    id: 23,
    category: '产后悦己',
    destination: '全职宝妈 (宝宝1岁4个月)',
    topic: '感觉很久没有阅读看书，脑子变笨，求阅读复健。',
    style: '身心塑形与产后悦己流',
    core: '方案：从轻阅读微信读书听书开始，每日睡前阅读10页精选书籍，加入宝妈共读社群打卡。'
  },
  {
    id: 24,
    category: '职场复工',
    destination: '初入职场复工宝妈',
    topic: '同事聚会无法参加，感觉脱离职场圈子。',
    style: '职场复工与平衡破局流',
    core: '方案：利用午餐时间与同事午餐交流信息，不在工作场合频繁抱怨育儿辛苦，保持专业开朗气场。'
  },
  {
    id: 25,
    category: '家庭和谐',
    destination: '全职宝妈 (宝宝2岁)',
    topic: '与公婆住在一起隐私空间少，求空间界限。',
    style: '温馨家庭与亲密关系流',
    core: '方案：明确主卧与儿童房为绝对私人领地，安排公婆定居或定期度假休息，给彼此呼吸感。'
  },
  {
    id: 26,
    category: '副业探索',
    destination: '摄影爱好者宝妈',
    topic: '想接单同城新生儿与儿童户外写真。',
    style: '宝妈副业与变现探索流',
    core: '方案：以自然光纪实风格积累自家宝宝写真样片，小红书同城种草招募互勉体验官，转化付费客单。'
  },
  {
    id: 27,
    category: '时间管理',
    destination: '全职宝妈 (宝宝9个月)',
    topic: '一天到晚抱着孩子手酸背痛，求放手辅助神器。',
    style: '时间管理与高效带娃流',
    core: '方案：引入人体工学腰凳、婴儿围栏与探索探索玩偶，建立阶段性独立自主游戏角。'
  },
  {
    id: 28,
    category: '产后悦己',
    destination: '全职宝妈 (宝宝2岁)',
    topic: '想培养一项终身爱好（如绘画、瑜伽、弹琴）。',
    style: '身心塑形与产后悦己流',
    core: '方案：每周报名一次线下零基础兴趣体验课，享受纯粹不带娃的2小时自我沉浸时光。'
  },
  {
    id: 29,
    category: '职场复工',
    destination: '自由职业转全职复工',
    topic: '适应高强度工作节奏与考核压力。',
    style: '职场复工与平衡破局流',
    core: '方案：建立工作 Task 清单与优先级分类，利用番茄工作法保持高度专注，产出干货成果。'
  },
  {
    id: 30,
    category: '家庭和谐',
    destination: '全职宝妈 (宝宝3岁准备幼升小)',
    topic: '面临幼小衔接焦虑，与丈夫教育理念分歧。',
    style: '温馨家庭与亲密关系流',
    core: '方案：统一“重视习惯与兴趣大于抢跑灌输”共识，分工协同（宝妈抓习惯，爸爸抓体育运动）。'
  }
];

const samples = ref<BaomaSample[]>(raw30Samples);

const filteredSamples = computed(() => {
  return samples.value.filter(s => {
    const matchCat = currentCategory.value === '全部' || s.category === currentCategory.value;
    const matchQuery = !searchQuery.value.trim() || 
      s.topic.includes(searchQuery.value) || 
      s.destination.includes(searchQuery.value) ||
      s.style.includes(searchQuery.value) || 
      s.core.includes(searchQuery.value);
    return matchCat && matchQuery;
  });
});

const totalPages = computed(() => Math.ceil(filteredSamples.value.length / pageSize) || 1);

const paginatedSamples = computed(() => {
  const start = (currentPage.value - 1) * pageSize;
  return filteredSamples.value.slice(start, start + pageSize);
});

watch([currentCategory, searchQuery], () => {
  currentPage.value = 1;
});
</script>

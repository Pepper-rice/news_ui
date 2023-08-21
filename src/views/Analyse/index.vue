<template>
    <div class='analyse'>
        <div class='wrap'>
            <div class='top'>
                {{ text }}
            </div>
            <div class='main-title'>
                <span class='dot-line-gray' />
                <span class='title'>分析结果</span>
            </div>
            <el-tabs v-model='currentTab' tab-position='left' type='border-card' style='min-height:500px;flex:1'>
                <el-tab-pane label='主题分析'>
                    <div style='display:flex;justify-content:center;height:200px;align-items:center'>
                        <el-radio-group v-model='subject' size='large'>
                            <el-radio-button v-for='c in chans' :key='c' :label='c' :disabled='subject != c' />
                        </el-radio-group>
                    </div>
                </el-tab-pane>
                <el-tab-pane label='摘要分析'>
                    <p v-for='s in summary' :key='s' style='margin-bottom:20px;color:#488fce;'>
                        {{ s }}
                    </p>
                </el-tab-pane>
                <el-tab-pane label='关键词分析' name='keywords'>
                    <vue-echarts v-if='currentTab == &apos;keywords&apos;' :option='wordcloud' style='heigth:500px;' />
                </el-tab-pane>
                <el-tab-pane label='情感分析' name='sentiment'>
                    <template>
                        <p style='text-align:center;display:flex;align-items:center;'>
                            <span style='margin-right:20px;'>
                                判断为:
                            </span>
                            <template v-if='sentiment'>
                                <el-tag v-if='sentiment[0] >= 0.5' effect='dark' type='success'>正面</el-tag>
                                <el-tag v-else effect='dark' type='danger'>负面</el-tag>
                            </template>
                        </p>
                    </template>
                    <vue-echarts v-if='currentTab == &apos;sentiment&apos;' :option='pie' style='heigth:300px;' />
                </el-tab-pane>

                <el-tab-pane label='词性标注'>
                    <div class='flex'>
                        <div style='max-width:80%;min-width: 80%;'>
                            <my-tag v-for='tag, idx in tags' :key='idx' :val='tag' />
                        </div>
                        <div>
                            <p style='margin-bottom: 22px;height: 45px;line-height: 45px;font-size: 16px;color: #979797;'>词性类别图示:</p>
                            <el-tag
                                v-for='value in tp'
                                :key='value.name'
                                style='margin-right: 10px; margin-top: 5px'
                                :color='value.color'
                                effect='dark'
                                type='info'
                                v-text='value.name'
                            />
                        </div>
                    </div>
                </el-tab-pane>

                <el-tab-pane label='手动标注'>
                    <div class='flex'>
                        <div
                            id='annotateContent'
                            style='max-width:80%;min-width: 80%;'
                            @mouseup.stop='highlight($event)'
                            v-html='text2'
                        />
                        <div>
                            <p style='margin-bottom: 22px;height: 45px;line-height: 45px;font-size: 16px;color: #979797;'>词性类别图示:</p>
                            <el-tag
                                v-for='value in tp'
                                :key='value.name'
                                style='margin-right: 10px; margin-top: 5px'
                                :color='value.color'
                                effect='dark'
                                type='info'
                                v-text='value.name'
                            />
                        </div>
                    </div>
                </el-tab-pane>

            </el-tabs>
        </div>

        <div
            v-show='labeldialog'
            class='labelbutton2'
        >
            <div
                v-for='item in labels'
                :key='item.color'
                @click='labelmenuchose(RandomId(),item)'
            >
                <span :style='{&apos;backgroundColor&apos;:item.color}' />
                <div>{{ item.label }}</div>
            </div>
        </div>
    </div>
</template>

<script>
import request from '/@/utils/request'
import MyTag from './my-tag.vue'
import { tp } from './my-tag.vue'
export default {
    components: {
        MyTag
    },
    props: ['subject'],
    data() {
        return {
            tp,
            chans: ['国内', '国际', '体育', '娱乐', '军事', '科技', '财经', '股市', '美股', '社会'],
            currentTab: 0,
            text: '',
            text2:'',
            pie: null,
            wordcloud: null,
            sentiment: null,
            summary: null,
            tags: [],
            textrender: true,
            sel: null,
            range: null,
            labeldialog: false,
            editbutton: false,
            labels: [
                { label: '人名', color: '#50c6fb' },
                { label: '地名', color: '#d07f7f' },
                { label: '组织', color: '#8e8b8b' },
                { label: '字符串', color: '#96dbf8' },
                { label: '专有名词', color: '#8bc3f5' },
                { label: '名词', color: '#488fce' },
                { label: '电话号码', color: '#118844' },
                { label: '时间词', color: '#c7aee7' },
                { label: '处所词', color: '#cee887' },
                { label: '方位词', color: '#c9aaca' },
                { label: '区别词', color: '#67a6d9' },
                { label: '状态词', color: '#f3e988' },
                { label: '代词', color: '#9acccd' },
                { label: '数词', color: '#986699' },
                { label: '量词', color: '#ff9899' },
                { label: '副词', color: '#ffcccb' },
                { label: '介词', color: '#99cc67' },
                { label: '连词', color: '#8ea4de' },
                { label: '网页链接', color: '#668800' },
                { label: '助词', color: '#4dd9e6' },
                { label: '电子邮件', color: '#669944' },
                { label: '叹词', color: '#d7acd7' },
                { label: '语气词', color: '#f48363' },
                { label: '拟声词', color: '#f2d404' },
                { label: '前缀', color: '#30aadd' },
                { label: '身份证号', color: '#669911' },
                { label: 'IP地址', color: '#884466' },
                { label: '后缀', color: '#880066' },
                { label: '标点符号', color: '#994466' },
                { label: '成语', color: '#ff33ff' },
                { label: '简称略语', color: '#1133cc' }
            ],
            textchose: ''
        }
    },
    mounted() {

        let text = localStorage.getItem('text')
        if (text) {
            this.text = text
            const text2 = JSON.parse(localStorage.getItem('text2'))
            const id = localStorage.getItem('id')
            this.text2 = text2 && text2[id] ? text2[id] : text
            this.analyse()
        }
    },
    methods: {

        async analyse() {
            let { data: sentiment } = await request.post('/sentiment/', { text: this.text })
            this.sentiment = sentiment

            let { data: pie } = await request.post('/pie/', { text: this.text })
            this.pie = pie

            let { data: tags } = await request.post('/tag/', { text: this.text })
            this.tags = tags

            let { data: wordcloud } = await request.post('/keywords/', { text: this.text })
            this.wordcloud = wordcloud

            let { data: summary } = await request.post('/summary/', { text: this.text })
            this.summary = summary

        },
        
        // 随机生成id
        RandomId() {
            const CHARCODE_A_LC = 97
            const lowercaseAlphabet = new Array(26)
                .fill(null)
                .map((v, i) => String.fromCharCode(CHARCODE_A_LC + i))

            let str = ''
            for (let i = 0; i <= 5; i++) {
                str += lowercaseAlphabet[Math.round(Math.random() * lowercaseAlphabet.length)]
            }

            return str

        },

        // 选中标注文本
        highlight(e) {
            if (!window.getSelection().toString()) {
                this.labeldialog = false // 选中文本后出现标注列表弹框
                return
            }

            this.range = window.getSelection().getRangeAt(0)
            this.textchose = window.getSelection().toString()

            const { pageX, pageY } = e
            // 计算选中文本在屏幕中的位置，然后弹出标签列表弹窗
            this.setBoxPosition(pageX, pageY)
        },

        // 计算标注标签栏弹出的位置
        setBoxPosition(X, Y) {
            let labellist = document.querySelector('.labelbutton2')
            const maxHeight = document.body.clientHeight
            let height = Y + 10
            if (maxHeight < height + 248) {
                height = height - 200
            }
            labellist.style.cssText = `height:248px;overflow:auto;width:200px;position:fixed;left:${X + 40}px;top:${height}px;background:#fff;box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.2), 0 6px 20px 0 rgba(0, 0, 0, 0.19);border-radius: 4px`
            // 弹出标签列表
            this.labeldialog = true
            this.editbutton = false

        },

        // 选择标注标签
        labelmenuchose(id, item) {

            // 修改标签
            if (this.editbutton) { // 这个是标注后单击标签，实现修改标签，不需要可以直接走else
                const target = document.getElementById(`${this.targetId}`)
                const outerText = target.getElementsByTagName('em')[0].innerHTML
                const close = document.createElement('i')
                close.innerHTML = 'x'
                let color = `${item.color}40`
                target.innerHTML = `<em style="background:${color}!important;font-style: normal;">${outerText}</em><button style="border:1px solid #fff;background:#fff;padding: 0 2px;border-radius: 2px;position:relative;vertical-align: super;background:${item.color}!important;color:#fff">${item.label} <i style='color:red !important;'>${close.innerHTML}</i></button>`
                this.labeldialog = false
                const text = document.getElementById('annotateContent')
                const text2 = JSON.parse(localStorage.getItem('text2'))
                const textId = localStorage.getItem('id')
                text2[textId] = text.innerHTML
                localStorage.setItem('text2',JSON.stringify(text2))
            } else {
                // 添加标签
                if (!this.textchose) { // 如果选中文本为空，直接返回
                    return
                }
                this.addMarking(id, item, this.range)// 标注时间
                this.editbutton = true
                this.targetId = id
                this.labeldialog = false
                const text = document.getElementById('annotateContent')
                const text2 = JSON.parse(localStorage.getItem('text2'))
                const textId = localStorage.getItem('id')
                text2[textId] = text.innerHTML
                localStorage.setItem('text2',JSON.stringify(text2))
            }

           

        },

        addMarking(id, item, range) {
           
            const span = document.createElement('span')
            span.className = 'highlight'
            span.setAttribute('id', id)
            const close = document.createElement('i')
            close.innerHTML = 'x'

            span.addEventListener('click', (e) => {
                if (e.target.localName === 'i') {
                    this.targetId = e.target.parentNode.parentNode.id
                    this.deleteById(this.targetId)
                    return
                }
                this.targetId = e.target.parentNode.id
                this.labeldialog = true
            })

            range.surroundContents(span) // 把指定节点插入选区的起始位置，然后把指定节点的内容替换为选区的内容。
            let color = `${item.color}40`
            span.innerHTML = `<em style="background:${color}!important;font-style: normal;">${span.innerHTML}</em><button style="border:1px solid #fff;background:#fff;padding: 0 2px;border-radius: 2px;position:relative;vertical-align: super;background:${item.color}!important;color:#fff">${item.label} <i style='color:red !important;'>${close.innerHTML}</i></button>`
            window.getSelection().removeAllRanges()
        },
        // 删除标记
        deleteById(id) {
            const target = document.getElementById(id)
            const { innerHTML } = target.firstChild
            const span = document.createElement('span')
            target.parentNode.insertBefore(span, target)
            span.outerHTML = innerHTML
            target.remove()

        }
    }
}
</script>

<style lang="postcss" scoped>
.analyse {
  display: flex;
  padding: 30px;
  justify-content: center;

.wrap {
  width: 80%;
  display: flex;
  flex-direction: column;

.top {
  border-radius: 5px;
  overflow: hidden;
  padding: 20px;
  background-color: rgb(246, 246, 246);
  box-shadow: #999 1px 1px 5px;
}

.main-title {
  position: relative;
  margin: 28px 0 22px 0;
  height: 32px;
  font-size: 24px;
  line-height: 32px;
  color: #565656;

.dot-line-gray {
  position: absolute;
  top: 15px;
  left: 0;
  right: 0;
  z-index: 0;
  height: 1px;
  border: 1px dashed #888;
}

.title {
  position: absolute;
  padding-right: 26px;
  background-color: #f0f2f5;
  z-index: 1;
}
}
}
}

.labelbutton2 > div {
  display: flex;
  height: 40px;
  align-items: center;
}

.labelbutton2 > div > span {
  display: block;
  width: 20px;
  height: 20px;
  border: 1px solid #cccccc;
  border-radius: 5px;
  margin: 0 10px;
}

.labelbutton2 > div:hover {
  background-color: aqua;
}
</style>
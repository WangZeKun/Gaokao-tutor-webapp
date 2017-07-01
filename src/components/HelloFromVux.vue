<template>
  <div>
    <div class="vux-demo">
      <img class="logo" src="../assets/vux_logo.png">
      <h1></h1>
    </div>
    <group title="基本信息">
      <x-input title="姓名" v-model="form.name" :max="5"></x-input>
      <selector title="性别"
                v-model="form.sex"
                :options="[{key: 0, value: '男'}, {key: 1, value: '女'}]"></selector>
      <x-input title="联系电话" type="number" v-model="form.telephone"></x-input>
      <x-input title="家庭住址" v-model="form.address"></x-input>
      <x-input title="学校" v-model="form.school"></x-input>
      <x-input title="年级" v-model="form.grade"></x-input>
      <x-input title="班级" v-model="form.class"></x-input>
    </group>
    <group title="成绩信息（请填写最近一次考试信息）">
      <selector title="文理科"
                v-model="form.type"
                :options="[{key: 1, value: '文科'}, {key: 2, value: '理科'}]"
                @on-change="change"
      ></selector>
      <div v-if="form.type>0">
        <divider>填写成绩请用“分数/满分”的格式填写</divider>
        <x-input
          v-for="(x,index) in data[form.type]"
          :title="x"
          :key="index"
          v-model="form.score[x]"
          :is-type="check"
        ></x-input>
      </div>
    </group>
    <group title="想要补习（陪优）的科目及原因">
      <x-textarea :rows="8" v-model='form.text'></x-textarea>
    </group>

    <x-button style="border-radius:99px;" plain type="primary" action-type="button" @click.native="submit">报名参加
    </x-button>
    <br/>
    <br/>
  </div>
</template>

<script>
  import { Group, Cell, XInput, Checker, CheckerItem, Selector, Grid, GridItem, Divider, XTextarea, XButton } from 'vux'

  export default {
    components: {
      Group,
      Cell,
      XInput,
      Checker,
      CheckerItem,
      Selector,
      Grid,
      GridItem,
      Divider,
      XTextarea,
      XButton
    },
    data () {
      return {
        // note: changing this line won't causes changes
        // with hot-reload because the reloaded component
        // preserves its current state and we are modifying
        // its initial state.
        form: {
          name: '',
          sex: '',
          telephone: '',
          address: '',
          school: '',
          class: '',
          grade: '',
          type: '',
          score: [],
          text: ''
        },
        data: {
          1: ['语文', '数学', '英语', '地理', '历史', '政治'],
          2: ['语文', '数学', '英语', '物理', '化学', '生物']
        }
      }
    },
    methods: {
      check (e) {
        let index = e.indexOf('/')
        if (index !== -1) {
          let s = Number(e.substring(0, index))
          let fs = Number(e.substring(index + 1, e.length))
          if (fs >= s) {
            return {valid: true}
          }
        }
        return {valid: false, msg: '输入错误'}
      },
      change () {
        console.log(this.form.type - 1)
        this.form.score = [
          {'语文': '', '数学': '', '英语': '', '地理': '', '历史': '', '政治': ''},
          {'语文': '', '数学': '', '英语': '', '物理': '', '化学': '', '生物': ''}
        ][this.form.type - 1]
      },
      submit () {
        console.log(this.form.score)
        let params = new URLSearchParams()
        let keys = Object.keys(this.form)
        if (this.form.telephone.length !== 13) {
          this.$vux.toast.show({
            text: '请检查输入！',
            type: 'cancel'
          })
        }
        for (let x of keys) {
          if (this.form[x] === '') {
            this.$vux.toast.show({
              text: '请检查输入！',
              type: 'cancel'
            })
            return
          }
          if (x === 'score') {
            let ks = Object.keys(this.form[x])
            for (let i of ks) {
              if (!this.check(this.form[x][i]).valid) {
                this.$vux.toast.show({
                  text: '请正确输入成绩！',
                  type: 'cancel'
                })
                return
              }
            }
            params.append(x, JSON.stringify(this.form[x]))
            continue
          }
          params.append(x, this.form[x])
        }
        this.$vux.loading.show({
          text: 'Loading'
        })
        this.$http.post('/api/join', params)
          .then(() => {
            this.$vux.loading.hide()
            this.$vux.toast.show({
              text: '提交成功！',
              type: 'success'
            })
            this.form = {
              name: '',
              sex: '',
              telephone: '',
              address: '',
              school: '',
              class: '',
              grade: '',
              type: '',
              score: [],
              text: ''
            }
          })
          .catch(err => {
            console.log(err)
            this.$vux.loading.hide()
            this.$vux.toast.show({
              text: '提交失败！',
              type: 'cancel'
            })
          })
      }
    }
  }
</script>

<style>
  .vux-demo {
    text-align: center;
  }

  .logo {
    width: 100px;
    height: 100px
  }

</style>

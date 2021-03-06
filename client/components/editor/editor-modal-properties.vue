<template lang='pug'>
  v-dialog(
    v-model='isShown'
    persistent
    width='1000'
    :fullscreen='$vuetify.breakpoint.smAndDown'
    )
    .dialog-header
      v-icon(color='white') mdi-tag-text-outline
      .subtitle-1.white--text.ml-3 {{$t('editor:props.pageProperties')}}
      v-spacer
      v-btn.mx-0(
        outlined
        dark
        @click.native='close'
        )
        v-icon(left) mdi-check
        span {{ $t('common:actions.ok') }}
    v-card(tile)
      v-tabs(color='white', background-color='blue darken-1', dark, centered)
        v-tab {{$t('editor:props.info')}}
        v-tab {{$t('editor:props.scheduling')}}
        v-tab(disabled) {{$t('editor:props.scripts')}}
        v-tab {{$t('editor:props.social')}}
        v-tab-item
          v-card-text.pt-5
            .overline.pb-5 {{$t('editor:props.pageInfo')}}
            v-text-field(
              ref='iptTitle'
              outlined
              :label='$t(`editor:props.title`)'
              counter='255'
              v-model='title'
              )
            v-text-field(
              outlined
              :label='$t(`editor:props.shortDescription`)'
              counter='255'
              v-model='description'
              persistent-hint
              :hint='$t(`editor:props.shortDescriptionHint`)'
              )
          v-divider
          v-card-text.grey.pt-5(:class='$vuetify.theme.dark ? `darken-3-d3` : `lighten-5`')
            .overline.pb-5 {{$t('editor:props.path')}}
            v-container.pa-0(fluid, grid-list-lg)
              v-layout(row, wrap)
                v-flex(xs12, md2)
                  v-select(
                    outlined
                    :label='$t(`editor:props.locale`)'
                    suffix='/'
                    :items='namespaces'
                    v-model='locale'
                    hide-details
                  )
                v-flex(xs12, md10)
                  v-text-field(
                    outlined
                    :label='$t(`editor:props.path`)'
                    append-icon='mdi-folder-search'
                    v-model='path'
                    :hint='$t(`editor:props.pathHint`)'
                    persistent-hint
                    @click:append='showPathSelector'
                    )
          v-divider
          v-card-text.grey.pt-5(:class='$vuetify.theme.dark ? `darken-3-d5` : `lighten-4`')
            .overline.pb-5 {{$t('editor:props.categorization')}}
            v-chip-group.radius-5.mb-5(column, v-if='tags && tags.length > 0')
              v-chip(
                v-for='tag of tags'
                :key='`tag-` + tag'
                close
                label
                color='teal'
                text-color='teal lighten-5'
                @click:close='removeTag(tag)'
                ) {{tag}}
            v-combobox(
              :label='$t(`editor:props.tags`)'
              outlined
              v-model='newTag'
              :hint='$t(`editor:props.tagsHint`)'
              :items='newTagSuggestions'
              :loading='$apollo.queries.newTagSuggestions.loading'
              persistent-hint
              hide-no-data
              :search-input.sync='newTagSearch'
              )
        v-tab-item
          v-card-text
            .overline.pb-5 {{$t('editor:props.publishState')}} #[v-chip.ml-3(label, color='grey', small, outlined).white--text coming soon]
            v-switch(
              :label='$t(`editor:props.publishToggle`)'
              v-model='isPublished'
              color='primary'
              :hint='$t(`editor:props.publishToggleHint`)'
              persistent-hint
              disabled
              inset
              )
          v-divider
          v-card-text.grey.pt-5(:class='$vuetify.theme.dark ? `darken-3-d3` : `lighten-5`')
            v-container.pa-0(fluid, grid-list-lg)
              v-row
                v-col(cols='6')
                  v-dialog(
                    ref='menuPublishStart'
                    :close-on-content-click='false'
                    v-model='isPublishStartShown'
                    :return-value.sync='publishStartDate'
                    width='460px'
                    :disabled='!isPublished || true'
                    )
                    template(v-slot:activator='{ on }')
                      v-text-field(
                        v-on='on'
                        :label='$t(`editor:props.publishStart`)'
                        v-model='publishStartDate'
                        prepend-icon='mdi-calendar-check'
                        readonly
                        outlined
                        clearable
                        :hint='$t(`editor:props.publishStartHint`)'
                        persistent-hint
                        :disabled='!isPublished || true'
                        )
                    v-date-picker(
                      v-model='publishStartDate'
                      :min='(new Date()).toISOString().substring(0, 10)'
                      color='primary'
                      reactive
                      scrollable
                      landscape
                      )
                      v-spacer
                      v-btn(
                        flat=''
                        color='primary'
                        @click='isPublishStartShown = false'
                        ) {{$t('common:actions.cancel')}}
                      v-btn(
                        flat=''
                        color='primary'
                        @click='$refs.menuPublishStart.save(publishStartDate)'
                        ) {{$t('common:actions.ok')}}
                v-col(cols='6')
                  v-dialog(
                    ref='menuPublishEnd'
                    :close-on-content-click='false'
                    v-model='isPublishEndShown'
                    :return-value.sync='publishEndDate'
                    width='460px'
                    :disabled='!isPublished || true'
                    )
                    template(v-slot:activator='{ on }')
                      v-text-field(
                        v-on='on'
                        :label='$t(`editor:props.publishEnd`)'
                        v-model='publishEndDate'
                        prepend-icon='mdi-calendar-remove'
                        readonly
                        outlined
                        clearable
                        :hint='$t(`editor:props.publishEndHint`)'
                        persistent-hint
                        :disabled='!isPublished || true'
                        )
                    v-date-picker(
                      v-model='publishEndDate'
                      :min='(new Date()).toISOString().substring(0, 10)'
                      color='primary'
                      reactive
                      scrollable
                      landscape
                      )
                      v-spacer
                      v-btn(
                        flat=''
                        color='primary'
                        @click='isPublishEndShown = false'
                        ) {{$t('common:actions.cancel')}}
                      v-btn(
                        flat=''
                        color='primary'
                        @click='$refs.menuPublishEnd.save(publishEndDate)'
                        ) {{$t('common:actions.ok')}}

        v-tab-item
          v-card-text
            .overline.pb-3 {{$t('editor:props.js')}}
            v-textarea(
              outlined
              rows='5'
              :hint='$t(`editor:props.jsHint`)'
              persistent-hint
            )
          v-divider
          v-card-text.grey.pt-5(:class='$vuetify.theme.dark ? `darken-3-d3` : `lighten-5`')
            .overline.pb-3 {{$t('editor:props.css')}}
            v-textarea(
              outlined
              rows='5'
              :hint='$t(`editor:props.cssHint`)'
              persistent-hint
            )

        v-tab-item
          v-card-text
            .overline.pb-5 {{$t('editor:props.socialFeatures')}} #[v-chip.ml-3(label, color='grey', small, outlined).white--text coming soon]
            v-switch(
              :label='$t(`editor:props.allowComments`)'
              v-model='isPublished'
              color='primary'
              :hint='$t(`editor:props.allowCommentsHint`)'
              persistent-hint
              disabled
              inset
              )
            v-switch(
              :label='$t(`editor:props.allowRatings`)'
              v-model='isPublished'
              color='primary'
              :hint='$t(`editor:props.allowRatingsHint`)'
              persistent-hint
              disabled
              inset
              )
            v-switch(
              :label='$t(`editor:props.displayAuthor`)'
              v-model='isPublished'
              color='primary'
              :hint='$t(`editor:props.displayAuthorHint`)'
              persistent-hint
              disabled
              inset
              )
            v-switch(
              :label='$t(`editor:props.displaySharingBar`)'
              v-model='isPublished'
              color='primary'
              :hint='$t(`editor:props.displaySharingBarHint`)'
              persistent-hint
              disabled
              inset
              )

    page-selector(:mode='pageSelectorMode', v-model='pageSelectorShown', :path='path', :locale='locale', :open-handler='setPath')
</template>

<script>
import _ from 'lodash'
import { sync, get } from 'vuex-pathify'
import gql from 'graphql-tag'

/* global siteLangs, siteConfig */

export default {
  props: {
    value: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      isPublishStartShown: false,
      isPublishEndShown: false,
      pageSelectorShown: false,
      namespaces: siteLangs.length ? siteLangs.map(ns => ns.code) : [siteConfig.lang],
      newTag: '',
      newTagSuggestions: [],
      newTagSearch: ''
    }
  },
  computed: {
    isShown: {
      get() { return this.value },
      set(val) { this.$emit('input', val) }
    },
    mode: get('editor/mode'),
    title: sync('page/title'),
    description: sync('page/description'),
    locale: sync('page/locale'),
    tags: sync('page/tags'),
    path: sync('page/path'),
    isPublished: sync('page/isPublished'),
    publishStartDate: sync('page/publishStartDate'),
    publishEndDate: sync('page/publishEndDate'),
    pageSelectorMode () {
      return (this.mode === 'create') ? 'create' : 'move'
    }
  },
  watch: {
    value(newValue, oldValue) {
      if (newValue) {
        _.delay(() => {
          this.$refs.iptTitle.focus()
          // this.$tours['editorPropertiesTour'].start()
        }, 500)
      }
    },
    newTag(newValue, oldValue) {
      const tagClean = _.trim(newValue || '').toLowerCase()
      if (tagClean && tagClean.length > 0) {
        if (!_.includes(this.tags, tagClean)) {
          this.tags = [...this.tags, tagClean]
        }
        this.$nextTick(() => {
          this.newTag = null
        })
      }
    }
  },
  methods: {
    removeTag (tag) {
      this.tags = _.without(this.tags, tag)
    },
    close() {
      this.isShown = false
    },
    showPathSelector() {
      this.pageSelectorShown = true
    },
    setPath({ path, locale }) {
      this.locale = locale
      this.path = path
    }
  },
  apollo: {
    newTagSuggestions: {
      query: gql`
        query ($query: String!) {
          pages {
            searchTags (query: $query)
          }
        }
      `,
      variables () {
        return {
          query: this.newTagSearch
        }
      },
      fetchPolicy: 'cache-first',
      update: (data) => _.get(data, 'pages.searchTags', []),
      skip () {
        return !this.value || _.isEmpty(this.newTagSearch)
      },
      throttle: 500
    }
  }
}
</script>

<style lang='scss'>

</style>

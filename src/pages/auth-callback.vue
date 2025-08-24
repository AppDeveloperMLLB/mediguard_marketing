<template>
  <v-container fluid class="fill-height">
    <v-row justify="center" align="center" class="fill-height">
      <v-col cols="12" sm="8" md="6" lg="4">
        <v-card class="pa-6 text-center" elevation="8">
          <v-card-title class="text-h4 mb-4">
            MediGuard
          </v-card-title>

          <!-- 認証処理中 -->
          <div v-if="authState === 'processing'" class="py-8">
            <v-progress-circular
              indeterminate
              color="primary"
              size="64"
              class="mb-4"
            />
            <v-card-text class="text-h6">
              認証処理中...
            </v-card-text>
          </div>

          <!-- 認証成功 -->
          <div v-else-if="authState === 'success'" class="py-8">
            <v-icon
              icon="mdi-check-circle"
              size="64"
              color="success"
              class="mb-4"
            />
            <v-card-text class="text-h6 text-success mb-6">
              ✓ メール認証が完了しました
            </v-card-text>
            
            <div class="mb-4">
              <v-btn
                color="primary"
                size="large"
                variant="elevated"
                @click="openApp"
                class="mb-3"
              >
                アプリで開く
              </v-btn>
            </div>
            
            <v-card-text class="text-body-2">
              アプリがインストールされていない場合は、<br>
              App Store / Google Playからダウンロードしてください。
            </v-card-text>
          </div>

          <!-- 認証エラー -->
          <div v-else-if="authState === 'error'" class="py-8">
            <v-icon
              icon="mdi-alert-circle"
              size="64"
              color="error"
              class="mb-4"
            />
            <v-card-text class="text-h6 text-error">
              認証エラーが発生しました
            </v-card-text>
          </div>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'

type AuthState = 'processing' | 'success' | 'error'

const authState = ref<AuthState>('processing')
let accessToken = ''
let refreshToken = ''

const processAuth = () => {
  const urlParams = new URLSearchParams(window.location.search)
  accessToken = urlParams.get('access_token') || ''
  refreshToken = urlParams.get('refresh_token') || ''
  const error = urlParams.get('error')

  if (error) {
    authState.value = 'error'
  } else if (accessToken) {
    authState.value = 'success'
    // 2秒後に自動的にアプリを開く試行
    setTimeout(() => {
      openApp()
    }, 2000)
  } else {
    authState.value = 'error'
  }
}

const openApp = () => {
  const appUrl = `mediguard://auth/success?access_token=${accessToken}&refresh_token=${refreshToken}`
  
  // アプリ起動を試行
  window.location.href = appUrl
  
  // アプリが開けない場合のフォールバック処理は自動的に行われる
  // (ブラウザがカスタムURLスキームを処理できない場合、何も起こらずWebページが表示されたまま)
}

onMounted(() => {
  processAuth()
})
</script>
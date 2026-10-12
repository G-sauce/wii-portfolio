<script setup lang="ts">
import { ref, onMounted, onUnmounted, computed } from 'vue'

// --- System State ---
const showBlackOverlay = ref(true)
const showWarningText = ref(true)
const playMainMusic = ref(false)

const currentTime = ref('7:05')
const currentAmPm = ref('PM')
const currentDate = ref('Fri 10/9')
const currentDateBoard = ref('Fri 10/9')
const activeChannelId = ref<string | null>(null)
const cursorPosition = ref({ x: 0, y: 0 })
const cursorVisible = ref(false)
const cursorPressed = ref(false)

// --- Settings & Preferences State ---
const showSettings = ref(false)
const settingsPage = ref(1)
const bgmVolume = ref(50)
const sfxVolume = ref(80)
const crtEnabled = ref(true)

let audioCtx: AudioContext | null = null
let timer: number

// --- Message Board State ---
const msgView = ref<'board' | 'read' | 'compose'>('board')
const activeMessage = ref<any>(null)
const messagePage = ref(0)
const composeName = ref('')
const composeContent = ref('')

const messages = ref([
  {
    id: 1,
    sender: 'Client',
    date: 'Today',
    content: 'Hi Azi, I just wanted to say that the 3D printed keychains you made for our event were a huge hit! Everyone loved them, and they were a great addition to our merchandise. Thank you for your excellent work!',
    color: 'bg-white'
  },
  {
    id: 2,
    sender: 'Achi',
    date: 'Yesterday',
    content: "Don't forget to track the filament consumption for the new keychain orders in the studio spreadsheet! Love, Achi",
    color: 'bg-[#fef9c3]'
  }
])

// --- Portfolio Data & Setup ---
const portfolioData = ref({
  name: 'Jasmine Angela C. Relucio',
  nickname: 'jazy',
  location: 'Cagayan, Philippines',
  degree: 'Bachelor of Science in Computer Engineering',
  school: 'Cagayan State University - Carig Campus',
  specialization: 'Hardware Engineering, Robotics & Mechatronics',
  studio: 'azi',
  framework: 'Vue 3 + Vite + Tailwind CSS',
  author: 'Jasmine Angela C. Relucio (jazy)',
  email: 'jasminerelucio74@gmail.com',
  address: 'Centro 07, Tuguegarao City, Cagayan, Philippines',
  github: 'https://github.com/G-sauce',
  linkedin: 'https://www.linkedin.com/in/jasminerelucio/'
})

// --- Channel Configuration ---
const channelConfigs = ref<Record<string, any>>({
  about: {
    id: 'about',
    title: 'About Azi',
    modalBg: 'from-[#ffedd5] via-[#fed7aa] to-[#fdba74]',
    textColor: 'text-[#7c2d12]',
    icon: 'person',
    gridBg: 'bg-[#fed7aa]',
    promptText: 'Press "Start" to download CV.',
    actionUrl: '/Relucio - Resume.pdf',
    hasStartButton: true,
    content: {
      p1: 'Description: Computer Engineering Student.',
      p2: `Contact Info: ${portfolioData.value.email}`,
      p3: `Address: ${portfolioData.value.address}`
    }
  },
  projects: {
    id: 'projects',
    title: 'Projects',
    modalBg: 'from-[#eff6ff] via-[#bfdbfe] to-[#60a5fa]',
    textColor: 'text-[#1e3a8a]',
    icon: '⭐',
    gridBg: 'bg-[#1d4ed8]',
    promptText: 'Press "Start" to view Project Repositories.',
    actionUrl: `${portfolioData.value.github}?tab=repositories`,
    hasStartButton: true,
    content: {
      p1: '• Sumobot Making for ITE Convention: Custom autonomous sumo robot designed with high-torque motor drivers and IR sensor logic for competition combat.',
      p2: '• 3D Print Order Tracker: Dedicated business & queue management system designed for azi 3D printing services.',
      p3: '• STREPAS: Standalone Tracking & Real-Time Edge-Based Proctoring Assistance System utilizing YOLO26 pose detection and ByteTrack tracking on Orange Pi 5.'
    }
  },
  experience: {
    id: 'experience',
    title: 'Experience',
    modalBg: 'from-[#fefce8] via-[#fef08a] to-[#fde047]',
    textColor: 'text-[#713f12]',
    icon: '💼',
    gridBg: 'bg-[#fef08a]',
    promptText: '',
    actionUrl: null,
    hasStartButton: false,
    content: {
      p1: '• Completed a 240-hour OJT Practicum at Blue Fox Summit Life Insurance Agency Inc. in Makati City.',
      p2: '• Working as a Freelance Hardware Technician on the side, diagnosing and repairing all types of gadgets, including custom console software and hardware modding.',
      p3: '• Founder & Operator of azi, managing Bambu Lab 3D printers, filament drying (PLA, PETG, ASA), and client fabrication queues.'
    }
  },
  skills: {
    id: 'skills',
    title: 'Skills & Tech',
    modalBg: 'from-[#f8fafc] via-[#e2e8f0] to-[#cbd5e1]',
    textColor: 'text-[#0f172a]',
    icon: '⚡',
    gridBg: 'bg-[#e2e8f0]',
    promptText: '',
    actionUrl: null,
    hasStartButton: false,
    content: {
      p1: '• Programming & Web: Python, PHP, JavaScript, C++, Vue 3, Tailwind CSS, HTML5/CSS3.',
      p2: '• AI & Edge Computing: YOLO26 pose detection, ByteTrack tracking, RKNN-Toolkit2 quantization on Rockchip RK3588S / Orange Pi 5.',
      p3: '• Tools & Databases: MySQL, Microsoft Access, XAMPP, Git, Figma, Canva, BeautifulSoup, Selenium.',
      p4: '• Hardware & Fabrication: Bambu Studio, OrcaSlicer, Soldering, Console Modding.'
    }
  },
  achievements: {
    id: 'achievements',
    title: 'Achievements',
    modalBg: 'from-[#fef3c7] via-[#fde68a] to-[#f59e0b]',
    textColor: 'text-[#78350f]',
    icon: '🏆',
    gridBg: 'bg-[#f59e0b]',
    promptText: '',
    actionUrl: null,
    hasStartButton: false,
    content: {
      p1: '• ITE Convention Robotics Representative: Successfully designed and deployed an autonomous Sumobot robot for regional competition.',
      p2: '• Lead Edge AI Researcher: Developed STREPAS edge proctoring system using custom homography and YOLO26 pose estimation.',
      p3: '• azi Founder: Built an active 3D printing and hardware fabrication business servicing local client orders.'
    }
  },
  certificates: {
    id: 'certificates',
    title: 'Certificates & Badges',
    modalBg: 'from-[#ccfbf1] via-[#99f6e4] to-[#14b8a6]',
    textColor: 'text-[#134e4a]',
    icon: '📜',
    gridBg: 'bg-[#14b8a6]',
    promptText: 'Press "Start" to inspect LinkedIn Badges & Credentials.',
    actionUrl: 'https://www.linkedin.com/in/jasminerelucio/details/certifications/',
    hasStartButton: true,
    content: {
      p1: '• Computer Engineering Practicum Certificate - Blue Fox Summit Life Insurance Agency Inc. (Makati City).',
      p2: '• Verified LinkedIn Technical Skill Badges & Software Engineering Credentials.',
      p3: '• Edge AI & Embedded Systems Workshop Certificates.'
    }
  },
  linkedin: {
    id: 'linkedin',
    title: 'LinkedIn',
    modalBg: 'from-[#dbeafe] via-[#93c5fd] to-[#3b82f6]',
    textColor: 'text-[#1e3a8a]',
    icon: 'in',
    gridBg: 'bg-[#0a66c2]',
    promptText: '',
    actionUrl: portfolioData.value.linkedin,
    hasStartButton: false,
    content: {}
  },
  github: {
    id: 'github',
    title: 'GitHub',
    modalBg: 'from-[#3f3f46] via-[#27272a] to-[#18181b]',
    textColor: 'text-[#f4f4f5]',
    icon: '🐱',
    gridBg: 'bg-[#24292e]',
    promptText: '',
    actionUrl: portfolioData.value.github,
    hasStartButton: false,
    content: {}
  }
})

const activeChannel = computed(() => {
  return activeChannelId.value ? channelConfigs.value[activeChannelId.value] : null
})

const MESSAGES_PER_PAGE = 6
const messagePageCount = computed(() => Math.max(1, Math.ceil(messages.value.length / MESSAGES_PER_PAGE)))
const visibleMessages = computed(() => {
  const start = messagePage.value * MESSAGES_PER_PAGE
  return messages.value.slice(start, start + MESSAGES_PER_PAGE)
})

const changeMessagePage = (direction: number) => {
  messagePage.value = (messagePage.value + direction + messagePageCount.value) % messagePageCount.value
  playBlip(direction > 0 ? 1000 : 850)
}

const getChannelActionUrl = (channelId: string | null): string | null => {
  if (channelId === 'github') return portfolioData.value.github
  if (channelId === 'linkedin') return portfolioData.value.linkedin
  if (channelId === 'certificates') return 'https://www.linkedin.com/in/jasminerelucio/details/certifications/'
  if (!channelId) return null
  return channelConfigs.value[channelId]?.actionUrl ?? null
}

// --- Audio Synthesizer & Volume Control ---
const initAudio = () => {
  if (!audioCtx) {
    const AudioContextClass = window.AudioContext || (window as unknown as { webkitAudioContext: typeof AudioContext }).webkitAudioContext
    audioCtx = new AudioContextClass()
  }
  if (audioCtx.state === 'suspended') audioCtx.resume()
}

const playBootChime = () => {
  if (!audioCtx || sfxVolume.value === 0) return
  const now = audioCtx.currentTime
  const osc1 = audioCtx.createOscillator()
  const osc2 = audioCtx.createOscillator()
  const gain = audioCtx.createGain()

  osc1.type = 'sine'
  osc2.type = 'triangle'
  osc1.frequency.setValueAtTime(440, now)
  osc1.frequency.setValueAtTime(880, now + 0.1)
  osc2.frequency.setValueAtTime(554.37, now)
  osc2.frequency.setValueAtTime(1108.73, now + 0.1)

  const vol = (sfxVolume.value / 100) * 0.08
  gain.gain.setValueAtTime(0, now)
  gain.gain.linearRampToValueAtTime(vol, now + 0.05)
  gain.gain.exponentialRampToValueAtTime(0.001, now + 1.5)

  osc1.connect(gain)
  osc2.connect(gain)
  gain.connect(audioCtx.destination)

  osc1.start(now)
  osc2.start(now)
  osc1.stop(now + 1.5)
  osc2.stop(now + 1.5)
}

const startExperience = () => {
  if (!showWarningText.value) return
  initAudio()
  showWarningText.value = false
  playBootChime()

  window.setTimeout(() => {
    showBlackOverlay.value = false
    playMainMusic.value = true
  }, 1000)
}

const playBlip = (freq = 800) => {
  try {
    if (!audioCtx || sfxVolume.value === 0) return
    const osc = audioCtx.createOscillator()
    const gain = audioCtx.createGain()

    osc.type = 'sine'
    osc.frequency.setValueAtTime(freq, audioCtx.currentTime)

    const vol = (sfxVolume.value / 100) * 0.05
    gain.gain.setValueAtTime(vol, audioCtx.currentTime)
    gain.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime + 0.08)

    osc.connect(gain)
    gain.connect(audioCtx.destination)

    osc.start()
    osc.stop(audioCtx.currentTime + 0.08)
  } catch (e) {}
}

const playSelectSound = () => {
  try {
    if (!audioCtx || sfxVolume.value === 0) return
    const now = audioCtx.currentTime
    const osc = audioCtx.createOscillator()
    const gain = audioCtx.createGain()

    osc.type = 'triangle'
    osc.frequency.setValueAtTime(523.25, now)
    osc.frequency.setValueAtTime(659.25, now + 0.08)
    osc.frequency.setValueAtTime(783.99, now + 0.16)

    const vol = (sfxVolume.value / 100) * 0.1
    gain.gain.setValueAtTime(vol, now)
    gain.gain.exponentialRampToValueAtTime(0.001, now + 0.35)

    osc.connect(gain)
    gain.connect(audioCtx.destination)

    osc.start(now)
    osc.stop(audioCtx.currentTime + 0.35)
  } catch (e) {}
}

// --- Time and Date Logic ---
const updateClock = () => {
  const now = new Date()
  let hours = now.getHours()
  const minutes = now.getMinutes().toString().padStart(2, '0')

  currentAmPm.value = hours >= 12 ? 'PM' : 'AM'
  hours = hours % 12 || 12
  currentTime.value = `${hours}:${minutes}`

  const days = ['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat']
  const month = now.getMonth() + 1
  const date = now.getDate()

  currentDate.value = `${days[now.getDay()]} ${month}/${date}`
  currentDateBoard.value = `${days[now.getDay()]} ${month}/${date}`
}

const updateCustomCursor = (event: PointerEvent) => {
  if (event.pointerType === 'touch') {
    cursorVisible.value = false
    return
  }

  cursorPosition.value = { x: event.clientX - 10, y: event.clientY - 3 }
  cursorVisible.value = true
}

const handleCursorPointerDown = (event: PointerEvent) => {
  updateCustomCursor(event)
  cursorPressed.value = event.pointerType !== 'touch'
}

const handleCursorPointerUp = () => {
  cursorPressed.value = false
}

const handleCursorPointerOut = (event: PointerEvent) => {
  if (!event.relatedTarget) {
    cursorVisible.value = false
    cursorPressed.value = false
  }
}

onMounted(() => {
  updateClock()
  timer = window.setInterval(updateClock, 1000)
  document.addEventListener('pointermove', updateCustomCursor)
  document.addEventListener('pointerdown', handleCursorPointerDown)
  document.addEventListener('pointerup', handleCursorPointerUp)
  document.addEventListener('pointerout', handleCursorPointerOut)
})

onUnmounted(() => {
  window.clearInterval(timer)
  document.removeEventListener('pointermove', updateCustomCursor)
  document.removeEventListener('pointerdown', handleCursorPointerDown)
  document.removeEventListener('pointerup', handleCursorPointerUp)
  document.removeEventListener('pointerout', handleCursorPointerOut)
  if (audioCtx) {
    audioCtx.close()
    audioCtx = null
  }
})

// --- Main Grid Channels ---
const gridChannels = [
  { id: 'about', type: 'about' },
  { id: 'projects', type: 'projects' },
  { id: 'experience', type: 'experience' },
  { id: 'skills', type: 'skills' },
  { id: 'achievements', type: 'achievements' },
  { id: 'certificates', type: 'certificates' },
  { id: 'linkedin', type: 'linkedin' },
  { id: 'github', type: 'github' },
  { id: 'empty1', type: 'empty' },
  { id: 'empty2', type: 'empty' },
  { id: 'empty3', type: 'empty' },
  { id: 'empty4', type: 'empty' }
]

const openChannel = (id: string, type: string) => {
  if (type === 'empty') return
  playSelectSound()
  
  if (id === 'github') {
    window.open(portfolioData.value.github, '_blank', 'noopener,noreferrer')
    return
  }
  if (id === 'linkedin') {
    window.open(portfolioData.value.linkedin, '_blank', 'noopener,noreferrer')
    return
  }

  activeChannelId.value = id
}

const navigateChannel = (direction: number) => {
  const availableChannels = gridChannels.filter(c => c.type !== 'empty' && c.id !== 'github' && c.id !== 'linkedin')
  if (!availableChannels.length) return
  const currentIndex = availableChannels.findIndex(c => c.id === activeChannelId.value)
  const nextIndex = (currentIndex + direction + availableChannels.length) % availableChannels.length
  activeChannelId.value = availableChannels[nextIndex].id
  playBlip(direction > 0 ? 1000 : 850)
}

const executeStartAction = () => {
  playSelectSound()
  const actionUrl = getChannelActionUrl(activeChannelId.value)
  if (actionUrl) {
    if (activeChannelId.value === 'about') {
      const link = document.createElement('a')
      link.href = actionUrl
      link.download = 'Jasmine_Angela_Relucio_Resume.pdf'
      document.body.appendChild(link)
      link.click()
      document.body.removeChild(link)
    } else {
      window.open(actionUrl, '_blank', 'noopener,noreferrer')
    }
  } else {
    alert(`Viewing details for ${activeChannel.value?.title}`)
  }
}

// --- Message Board Actions ---
const openMessage = (message: any) => {
  playSelectSound()
  activeMessage.value = message
  msgView.value = 'read'
}

const sendMessage = () => {
  playSelectSound()
  if (composeName.value.trim() && composeContent.value.trim()) {
    messages.value.unshift({
      id: Date.now(),
      sender: composeName.value,
      date: 'Just now',
      content: composeContent.value,
      color: 'bg-white'
    })
    messagePage.value = 0
    composeName.value = ''
    composeContent.value = ''
    alert('Message pinned to the board!')
    msgView.value = 'board'
  }
}
</script>

<template>
  <div class="min-h-screen bg-[#e8ecef] font-sans select-none overflow-x-hidden flex flex-col relative wii-cursor-container">

    <!-- Custom Cursor using Untitled design (16).png -->
    <div
      v-if="cursorVisible"
      class="wii-custom-cursor"
      :class="{ 'wii-custom-cursor--pressed': cursorPressed }"
      :style="{ left: `${cursorPosition.x}px`, top: `${cursorPosition.y}px` }"
      aria-hidden="true"
    >
      <img src="/Untitled design (16).png" alt="Cursor" class="w-full h-full object-contain pointer-events-none" />
    </div>

    <component :is="'style'">
      @import url('https://fonts.googleapis.com/css2?family=Share+Tech+Mono&family=Georgia&display=swap');
    </component>

    <!-- CRT Scanlines Overlay -->
    <div v-if="crtEnabled" class="crt-scanlines-topmost"></div>

    <!-- Startup Health & Safety Overlay -->
    <transition name="fade-slow">
      <div v-if="showBlackOverlay" class="fixed inset-0 bg-black z-[100] flex items-center justify-center cursor-pointer" @click="startExperience">
        <transition name="fade">
          <div v-if="showWarningText" class="flex flex-col items-center justify-center font-sans text-center text-white px-4 w-full h-full">
            <div class="flex items-center justify-center gap-4 mb-10 w-full max-w-4xl border-b-2 border-gray-600 pb-4">
              <span class="text-yellow-400 text-5xl pb-2">⚠️</span>
              <h1 class="text-3xl md:text-4xl font-bold tracking-widest text-gray-100">WARNING-HEALTH AND SAFETY</h1>
            </div>
            <p class="text-lg md:text-xl mb-10 max-w-4xl leading-relaxed uppercase font-semibold text-gray-200">
              Hi, my name is <span class="italic">{{ portfolioData.nickname }}</span>.<br />
              Computer Engineering Student
            </p>
            <p class="text-lg md:text-xl mb-12 max-w-4xl leading-relaxed uppercase font-semibold text-gray-200">
              This is my portfolio inspired by the Wii main menu.<br />
              It works on desktop, tablet, and mobile.
            </p>
            <p class="text-xl animate-pulse flex items-center justify-center gap-2 font-medium mt-8">
              Click Here <span class="text-2xl border-2 border-white rounded px-2 py-0.5 shadow-[2px_2px_0px_white]">🖱️</span> to continue
            </p>
          </div>
        </transition>
      </div>
    </transition>

    <!-- Background Music -->
    <iframe
      v-if="playMainMusic"
      :src="`https://www.youtube.com/embed/5-E_0uhPzaE?autoplay=1&loop=1&playlist=5-E_0uhPzaE&volume=${bgmVolume}`"
      class="hidden w-0 h-0 opacity-0 absolute pointer-events-none"
      allow="autoplay; encrypted-media"
      title="Background music"
    ></iframe>

    <!-- Main Wii Channel Grid -->
    <main
      v-show="activeChannelId !== 'messageboard' && !activeChannel && !showSettings"
      class="w-full max-w-[1560px] mx-auto grid grid-cols-2 sm:grid-cols-3 lg:grid-cols-4 gap-3 sm:gap-4 md:gap-6 p-3 sm:p-4 md:p-5 z-10 flex-grow content-start pt-4 md:pt-5 pb-[190px] sm:pb-[220px] md:pb-[260px]"
    >
      <div
        v-for="channel in gridChannels"
        :key="channel.id"
        @mouseenter="channel.type !== 'empty' && playBlip(1000)"
        @click="openChannel(channel.id, channel.type)"
        class="relative flex flex-col items-center group cursor-pointer"
      >
        <div
          class="wii-channel-card w-full aspect-[2.18/1] relative rounded-[22px] shadow-[0_4px_10px_rgba(0,0,0,0.18)] overflow-hidden flex items-center justify-center wii-crt-channel"
          :class="[
            channelConfigs[channel.id]?.gridBg || 'bg-white',
            channel.type === 'empty' ? 'wii-empty-channel cursor-default' : 'active:scale-[0.98]'
          ]"
        >
          <div class="absolute inset-0 rounded-[22px] border-2 border-transparent group-hover:border-[#38bdf8] group-hover:shadow-[0_0_15px_rgba(56,189,248,0.5)] transition-all duration-200 pointer-events-none z-30"></div>

          <!-- About channel avatar -->
          <div v-if="channel.id === 'about'" class="absolute inset-0 flex items-center justify-center z-20">
            <div class="w-16 h-16 rounded-full bg-slate-200 border-2 border-slate-400 flex items-center justify-center shadow-sm overflow-hidden">
              <img src="/image_528537.png" alt="About Azi" class="w-full h-full object-cover rounded-full" />
            </div>
          </div>
          <div v-else-if="channel.id === 'projects'" class="absolute inset-0 flex items-center justify-center z-20 text-[#facc15]">
            <svg class="w-20 h-20 drop-shadow-md" fill="currentColor" viewBox="0 0 24 24">
              <path d="M12 17.27L18.18 21l-1.64-7.03L22 9.24l-7.19-.61L12 2 9.19 8.63 2 9.24l5.46 4.73L5.82 21z"/>
            </svg>
          </div>
          <div v-else-if="channel.id === 'experience'" class="absolute inset-0 flex items-center justify-center z-20 text-black">
            <svg class="w-20 h-20 drop-shadow-sm" fill="currentColor" viewBox="0 0 24 24">
              <path d="M20 6h-4V4c0-1.11-.89-2-2-2h-4c-1.11 0-2 .89-2 2v2H4c-1.11 0-1.99.89-1.99 2L2 19c0 1.11.89 2 2 2h16c1.11 0 2-.89 2-2V8c0-1.11-.89-2-2-2zm-6 0h-4V4h4v2z"/>
            </svg>
          </div>
          <div v-else-if="channel.id === 'skills'" class="absolute inset-0 flex items-center justify-center z-20">
            <svg class="w-20 h-20 drop-shadow-sm" viewBox="0 0 128 128">
              <path fill="#3776AB" d="M63.3 2c-30.8 0-28.9 13.4-28.9 13.4l.1 13.9h29.4v4.2H23.1S2 31.8 2 62.9c0 31.1 18.2 30 18.2 30h10.9V77.1s-.6-18.2 17.9-18.2h30.8s17.1.2 17.1-16.6V25.2S98.8 2 63.3 2zm-15.6 9.4c2.8 0 5 2.3 5 5.1 0 2.8-2.3 5-5 5-2.8 0-5.1-2.2-5.1-5 0-2.8 2.3-5.1 5.1-5.1z"/>
              <path fill="#FFD43B" d="M64.7 126c30.8 0 28.9-13.4 28.9-13.4l-.1-13.9H64.1v-4.2h40.8s21.1 1.7 21.1-29.4c0-31.1-18.2-30-18.2-30H96.9v15.8s.6 18.2-17.9 18.2H48.2s-17.1-.2-17.1 16.6v17.1S29.2 126 64.7 126zm15.6-9.4c-2.8 0-5-2.3-5-5.1 0-2.8-2.3 5-5 5 2.8 0 5.1 2.2 5.1 5 0 2.8-2.3 5.1-5.1 5.1z"/>
            </svg>
          </div>
          <div v-else-if="channel.id === 'achievements'" class="absolute inset-0 flex items-center justify-center z-20 text-white">
            <span class="text-5xl">🏆</span>
          </div>
          <div v-else-if="channel.id === 'certificates'" class="absolute inset-0 flex items-center justify-center z-20 text-white">
            <span class="text-5xl">📜</span>
          </div>
          <div v-else-if="channel.id === 'linkedin'" class="absolute inset-0 flex items-center justify-center z-20 text-white">
            <span class="text-6xl font-extrabold tracking-tighter">in</span>
          </div>
          <div v-else-if="channel.id === 'github'" class="absolute inset-0 flex items-center justify-center z-20 text-white">
            <svg class="w-20 h-20" fill="currentColor" viewBox="0 0 24 24">
              <path fill-rule="evenodd" clip-rule="evenodd" d="M12 2C6.477 2 2 6.484 2 12.017c0 4.425 2.865 8.18 6.839 9.504.5.092.682-.217.682-.483 0-.237-.008-.868-.013-1.703-2.782.605-3.369-1.343-3.369-1.343-.454-1.158-1.11-1.466-1.11-1.466-.908-.62.069-.608.069-.608 1.003.07 1.53 1.032 1.53 1.032.892 1.53 2.341 1.088 2.91.832.092-.647.35-1.088.636-1.338-2.22-.253-4.555-1.113-4.555-4.951 0-1.093.39-1.988 1.029-2.688-.103-.253-.446-1.272.098-2.65 0 0 .84-.27 2.75 1.026A9.564 9.564 0 0112 6.844c.85.004 1.705.115 2.504.337 1.909-1.296 2.747-1.027 2.747-1.027.546 1.379.202 2.398.1 2.651.64.7 1.028 1.595 1.028 2.688 0 3.848-2.339 4.695-4.566 4.943.359.309.678.92.678 1.855 0 1.338-.012 2.419-.012 2.747 0 .268.18.58.688.482A10.019 10.019 0 0022 12.017C22 6.484 17.522 2 12 2z"/>
            </svg>
          </div>

          <div v-if="channel.type === 'empty'" class="absolute inset-0 flex items-center justify-center pointer-events-none opacity-[0.25]">
            <span class="text-3xl font-black text-[#9ca3af] tracking-widest font-sans">Wii</span>
          </div>
        </div>

        <div v-if="channel.type !== 'empty'" class="absolute top-full left-1/2 -translate-x-1/2 mt-2 z-40 pointer-events-none">
          <span class="bg-white/95 px-3 py-0.5 rounded-full text-[11px] font-sans font-bold text-gray-800 shadow-md border border-gray-200 opacity-0 group-hover:opacity-100 transition-opacity duration-200 whitespace-nowrap block">
            {{ channelConfigs[channel.id]?.title }}
          </span>
        </div>
      </div>
    </main>

    <!-- Homepage Footer -->
    <div
      v-show="activeChannelId !== 'messageboard' && !activeChannel && !showSettings"
      class="wii-home-footer fixed bottom-0 left-0 right-0 h-[170px] sm:h-[195px] md:h-[220px] lg:h-[235px] z-30 pointer-events-none"
    >
      <div class="relative w-full h-full flex items-end justify-between px-3 sm:px-6 md:px-16 pb-0 pointer-events-auto">
        <svg class="absolute inset-0 w-full h-full overflow-visible pointer-events-none z-0" viewBox="0 0 1000 235" preserveAspectRatio="none">
          <defs>
            <linearGradient id="wiiFooterSide" x1="0" y1="0" x2="1" y2="1">
              <stop offset="0" stop-color="#d1d5db" />
              <stop offset="0.55" stop-color="#aeb6c0" />
              <stop offset="1" stop-color="#c5cad1" />
            </linearGradient>
          </defs>
          <path d="M-50 0 C205 0 230 96 365 96 L635 96 C770 96 795 0 1050 0 L1050 235 L-50 235 Z" fill="url(#wiiFooterSide)" opacity=".98" />
          <path d="M0 0 C210 0 230 95 365 95 L635 95 C770 95 790 0 1000 0" fill="none" stroke="#38bdf8" stroke-width="4" />
        </svg>

        <!-- Wii Settings Button -->
        <div class="footer-control relative z-10 flex items-center justify-center mb-4 sm:mb-6 md:mb-10">
          <div class="footer-button-plate footer-button-plate-left absolute -left-2 sm:-left-2 md:-left-3 w-[88px] h-[88px] sm:w-[120px] sm:h-[120px] md:w-[154px] md:h-[154px] bg-[#aeb6c0] rounded-full border border-white/55 z-0 pointer-events-none"></div>
          <div class="footer-button-shell overflow-hidden w-[68px] h-[68px] sm:w-[100px] sm:h-[100px] md:w-[132px] md:h-[132px] rounded-full bg-gradient-to-b from-[#f8fafc] to-[#cbd5e1] border-[4px] border-[#38bdf8] shadow-[0_5px_12px_rgba(55,65,81,0.18)] flex items-center justify-center relative z-10">
            <button
              type="button"
              @click="showSettings = true; settingsPage = 1; playBlip(600)"
              class="footer-round-button w-full h-full rounded-full bg-gradient-to-b from-white to-[#e2e8f0] flex items-center justify-center text-[#718096] font-black text-3xl md:text-4xl hover:text-[#00a3e0]"
            >
              Wii
            </button>
          </div>
        </div>

        <!-- Clock & Date -->
        <div class="absolute left-1/2 top-0 -translate-x-1/2 z-20 w-full flex flex-col items-center pointer-events-none">
          <div class="h-[78px] md:h-[84px] flex items-end justify-center gap-1.5 md:gap-2 pb-1 drop-shadow-sm">
            <template v-for="(char, index) in currentTime" :key="`clock-${index}`">
              <span v-if="char === ':'" class="relative inline-block w-[10px] md:w-[13px] h-[44px] md:h-[58px]">
                <span class="absolute top-[14px] left-1/2 -translate-x-1/2 w-[6px] h-[6px] bg-[#9ca3af] rounded-[1px]"></span>
                <span class="absolute bottom-[14px] left-1/2 -translate-x-1/2 w-[6px] h-[6px] bg-[#9ca3af] rounded-[1px]"></span>
              </span>
              <svg v-else class="w-[29px] h-[49px] md:w-[41px] md:h-[65px] overflow-visible" viewBox="0 0 40 68">
                <g fill="#9ca3af">
                  <polygon v-if="'02356789'.includes(char)" points="8,2 32,2 36,6 31,10 9,10 4,6" />
                  <polygon v-if="'045689'.includes(char)" points="4,8 9,12 9,29 4,33 0,29 0,12" />
                  <polygon v-if="'01234789'.includes(char)" points="36,8 40,12 40,29 36,33 31,29 31,12" />
                  <polygon v-if="'2345689'.includes(char)" points="8,30 32,30 36,34 31,38 9,38 4,34" />
                  <polygon v-if="'0268'.includes(char)" points="4,36 9,40 9,57 4,61 0,57 0,40" />
                  <polygon v-if="'013456789'.includes(char)" points="36,36 40,40 40,57 36,61 31,57 31,40" />
                  <polygon v-if="'023568'.includes(char)" points="8,58 32,58 36,62 31,66 9,66 4,62" />
                </g>
              </svg>
            </template>
            <span class="text-base md:text-[24px] font-bold text-[#9ca3af] font-sans self-end pb-1 ml-0.5">{{ currentAmPm }}</span>
          </div>
          <div class="absolute top-[110px] md:top-[112px] text-3xl md:text-[54px] font-bold text-[#6b7280] font-sans tracking-tight leading-none whitespace-nowrap">
            {{ currentDate }}
          </div>
        </div>

        <!-- Message Board Mail Button -->
        <div class="footer-control relative z-10 flex items-center justify-center mb-4 sm:mb-6 md:mb-10">
          <div class="footer-button-plate footer-button-plate-right absolute -right-2 sm:-right-2 md:-right-3 w-[88px] h-[88px] sm:w-[120px] sm:h-[120px] md:w-[154px] md:h-[154px] bg-[#aeb6c0] rounded-full border border-white/55 z-0 pointer-events-none"></div>
          <div class="footer-button-shell overflow-hidden w-[68px] h-[68px] sm:w-[100px] sm:h-[100px] md:w-[132px] md:h-[132px] rounded-full bg-gradient-to-b from-[#f8fafc] to-[#cbd5e1] border-[4px] border-[#38bdf8] shadow-[0_5px_12px_rgba(55,65,81,0.18)] flex items-center justify-center relative z-10">
            <button
              type="button"
              @click="activeChannelId = 'messageboard'; msgView = 'board'; playSelectSound()"
              class="footer-round-button overflow-hidden w-full h-full rounded-full bg-gradient-to-b from-white to-[#e2e8f0] flex items-center justify-center text-[#718096] hover:text-[#00a3e0]"
            >
              <svg class="w-14 h-14 md:w-16 md:h-16 fill-current" viewBox="0 0 24 24">
                <path d="M20 4H4c-1.1 0-1.99.9-1.99 2L2 18c0 1.1.9 2 2 2h16c1.1 0 2-.9 2-2V6c0-1.1-.9-2-2-2zm0 4l-8 5-8-5V6l8 5 8-5v2z"/>
              </svg>
            </button>
          </div>
        </div>
      </div>
    </div>

    <!-- WII SYSTEM SETTINGS MENU -->
    <transition name="fade">
      <div v-if="showSettings" class="fixed inset-0 z-[90] bg-[#111111] flex flex-col justify-between p-6 md:p-10 wii-cursor-container">
        
        <div class="flex items-center justify-between border-b-2 border-gray-700 pb-4">
          <div class="bg-gradient-to-r from-gray-200 to-gray-400 text-gray-900 px-6 py-2 rounded-lg font-bold text-xl md:text-2xl shadow-md">
            Wii System Settings {{ settingsPage }}
          </div>
          <div class="text-gray-400 font-mono text-lg font-bold">Ver. 3.4U</div>
        </div>

        <div class="flex-grow flex flex-col items-center justify-center gap-4 max-w-2xl mx-auto w-full">
          
          <template v-if="settingsPage === 1">
            <div @click="playSelectSound()" class="w-full bg-gradient-to-b from-white to-gray-200 border-2 border-[#38bdf8] rounded-full py-4 px-8 text-center text-gray-800 font-bold text-xl shadow-lg cursor-default flex justify-between items-center">
              <span>Console Nickname:</span>
              <span class="text-gray-600 font-normal px-2">{{ portfolioData.nickname }}</span>
            </div>
            <div @click="playSelectSound()" class="w-full bg-gradient-to-b from-white to-gray-200 border-2 border-[#38bdf8] rounded-full py-4 px-8 text-center text-gray-800 font-bold text-xl shadow-lg cursor-default flex justify-between items-center">
              <span>Studio:</span>
              <span class="text-gray-600 font-normal px-2">{{ portfolioData.studio }}</span>
            </div>
            <div @click="settingsPage = 2; playBlip(700)" class="w-full bg-gradient-to-b from-white to-gray-200 border-2 border-[#38bdf8] rounded-full py-4 px-8 text-center text-gray-800 font-bold text-xl shadow-lg hover:scale-[1.02] cursor-pointer">
              Sound Settings (BGM &amp; SFX Volumes) 🔊
            </div>
            <div @click="settingsPage = 3; playBlip(700)" class="w-full bg-gradient-to-b from-white to-gray-200 border-2 border-[#38bdf8] rounded-full py-4 px-8 text-center text-gray-800 font-bold text-xl shadow-lg hover:scale-[1.02] cursor-pointer">
              Screen &amp; Portfolio Setup (CRT &amp; Framework) 📺
            </div>
          </template>

          <template v-if="settingsPage === 2">
            <div class="w-full bg-white/10 p-6 rounded-2xl border border-gray-700 text-white space-y-6">
              <h3 class="text-2xl font-bold text-cyan-400 mb-2">Audio Volume Controls</h3>
              <div>
                <label class="block text-sm font-semibold mb-2">Background Music Volume: {{ bgmVolume }}%</label>
                <input v-model.number="bgmVolume" type="range" min="0" max="100" class="w-full accent-cyan-400">
              </div>
              <div>
                <label class="block text-sm font-semibold mb-2">Sound Effects Volume: {{ sfxVolume }}%</label>
                <input v-model.number="sfxVolume" type="range" min="0" max="100" class="w-full accent-cyan-400" @input="playBlip(900)">
              </div>
            </div>
          </template>

          <template v-if="settingsPage === 3">
            <div class="w-full bg-white/10 p-6 rounded-2xl border border-gray-700 text-white space-y-4">
              <h3 class="text-xl font-bold text-cyan-400">Screen &amp; System Setup</h3>
              <div class="flex items-center justify-between">
                <span class="font-semibold">CRT Scan Lines:</span>
                <button @click="crtEnabled = !crtEnabled; playBlip(800)" class="px-6 py-2 rounded-full font-bold text-sm" :class="crtEnabled ? 'bg-cyan-500 text-white' : 'bg-gray-600 text-gray-300'">
                  {{ crtEnabled ? 'ON (Soft Wii Scanlines)' : 'OFF' }}
                </button>
              </div>
              <div>
                <span class="block text-xs font-semibold mb-1 text-gray-300">Portfolio Framework</span>
                <div class="w-full border rounded p-2 text-gray-300 bg-gray-800/80 text-sm font-mono">
                  {{ portfolioData.framework }}
                </div>
              </div>
              <div>
                <span class="block text-xs font-semibold mb-1 text-gray-300">Author</span>
                <div class="w-full border rounded p-2 text-gray-300 bg-gray-800/80 text-sm font-mono">
                  {{ portfolioData.author }}
                </div>
              </div>
            </div>
          </template>

        </div>

        <div class="flex items-center justify-between pt-4 border-t-2 border-gray-700">
          <button @click="showSettings = false; playBlip(400)" class="w-48 py-3 bg-gradient-to-b from-white to-gray-300 rounded-full text-gray-800 font-bold text-xl shadow-md hover:bg-gray-100">
            Back
          </button>
          
          <div class="flex gap-2">
            <button @click="settingsPage = 1; playBlip(500)" class="w-12 h-12 rounded-lg font-bold text-lg flex items-center justify-center transition-colors" :class="settingsPage === 1 ? 'bg-white text-gray-900 shadow' : 'bg-gray-700 text-gray-400'">1</button>
            <button @click="settingsPage = 2; playBlip(500)" class="w-12 h-12 rounded-lg font-bold text-lg flex items-center justify-center transition-colors" :class="settingsPage === 2 ? 'bg-white text-gray-900 shadow' : 'bg-gray-700 text-gray-400'">2</button>
            <button @click="settingsPage = 3; playBlip(500)" class="w-12 h-12 rounded-lg font-bold text-lg flex items-center justify-center transition-colors" :class="settingsPage === 3 ? 'bg-white text-gray-900 shadow' : 'bg-gray-700 text-gray-400'">3</button>
          </div>
        </div>

      </div>
    </transition>

    <!-- Wii Channel Preview Modal -->
    <transition name="fade">
      <div v-if="activeChannel" class="fixed inset-0 z-[80] bg-transparent flex items-center justify-center p-3 md:p-4 wii-cursor-container">
        <section class="wii-channel-shell relative w-full h-full max-h-[720px] max-w-[1500px] overflow-hidden rounded-[34px] md:rounded-[48px] border border-white/70 bg-transparent flex flex-col shadow-[0_18px_45px_rgba(0,0,0,0.22)]">
          <div class="wii-channel-display relative flex-1 min-h-0 flex items-center justify-center px-12 sm:px-20 md:px-28" :class="[activeChannel.modalBg ? `bg-gradient-to-br ${activeChannel.modalBg}` : 'bg-transparent']">
            <div class="wii-channel-glass absolute inset-0 pointer-events-none"></div>

            <button type="button" @click="navigateChannel(-1)" class="absolute left-4 md:left-6 top-1/2 -translate-y-1/2 z-20 w-10 h-14 md:w-12 md:h-16 flex items-center justify-center hover:scale-110">
              <svg viewBox="0 0 40 64" class="w-full h-full drop-shadow-md">
                <path d="M34 2 Q37 0 35 7 L29 27 Q28 31 29 35 L35 57 Q37 63 32 60 L3 35 Q0 32 3 29 Z" fill="#54caff" stroke="#62bce9" stroke-width="1.5" />
              </svg>
            </button>

            <div class="relative z-10 w-full h-full flex flex-col items-center justify-center text-center p-4 overflow-y-auto">
              <h2 class="mb-4 text-3xl font-extrabold tracking-wide text-white drop-shadow-md md:text-5xl">{{ activeChannel.title }}</h2>
              
              <!-- Preview Screen Text Content -->
              <div class="my-2 max-w-3xl space-y-3 rounded-2xl border border-white/40 bg-black/30 p-5 text-left text-base font-semibold leading-relaxed text-white/95 shadow-lg backdrop-blur-md md:p-8 md:text-xl">
                <p v-if="activeChannel.content?.p1">{{ activeChannel.content.p1 }}</p>
                <p v-if="activeChannel.content?.p2">{{ activeChannel.content.p2 }}</p>
                <p v-if="activeChannel.content?.p3">{{ activeChannel.content.p3 }}</p>
                <p v-if="activeChannel.content?.p4">{{ activeChannel.content.p4 }}</p>
              </div>

              <p v-if="activeChannel.promptText" class="text-white/90 text-sm md:text-base mt-3 font-bold tracking-wider uppercase drop-shadow">{{ activeChannel.promptText }}</p>
            </div>

            <button type="button" @click="navigateChannel(1)" class="absolute right-4 md:right-6 top-1/2 -translate-y-1/2 z-20 w-10 h-14 md:w-12 md:h-16 flex items-center justify-center hover:scale-110">
              <svg viewBox="0 0 40 64" class="w-full h-full drop-shadow-md">
                <path d="M6 2 Q3 0 5 7 L11 27 Q12 31 11 35 L5 57 Q3 63 8 60 L37 35 Q40 32 37 29 Z" fill="#54caff" stroke="#62bce9" stroke-width="1.5" />
              </svg>
            </button>
          </div>

          <div class="wii-channel-footer h-[125px] md:h-[150px] shrink-0 border-t border-white/80 wii-empty-channel flex items-center justify-center gap-5 md:gap-12 px-4 md:px-10">
            <button type="button" @click="activeChannelId = null; playBlip(400)" class="w-[42%] max-w-[340px] h-[58px] md:h-[92px] rounded-full bg-gradient-to-b from-white to-[#d7dfe9] border-[3px] border-[#55c9f4] text-[#414141] text-xl md:text-[40px] font-normal shadow-md hover:scale-105 transition-all">
              Wii Menu
            </button>
            <button v-if="activeChannel.hasStartButton" type="button" @click="executeStartAction" class="w-[42%] max-w-[340px] h-[58px] md:h-[92px] rounded-full bg-gradient-to-b from-white to-[#d7dfe9] border-[3px] border-[#55c9f4] text-[#222] text-xl md:text-[40px] font-normal shadow-md hover:scale-105 transition-all">
              Start
            </button>
          </div>
        </section>
      </div>
    </transition>

    <!-- Message Board View -->
    <transition name="fade">
      <div v-if="activeChannelId === 'messageboard'" class="wii-message-board fixed inset-0 z-[80] flex flex-col wii-cursor-container">
        
        <!-- Board Main View -->
        <div v-if="msgView === 'board'" class="relative flex flex-1 items-center justify-center px-12 sm:px-20 pb-28 sm:pb-36">
          
          <!-- Left Navigation Arrow -->
          <button
            type="button"
            @click="changeMessagePage(-1)"
            aria-label="Previous messages"
            class="absolute left-4 sm:left-10 top-1/2 -translate-y-1/2 z-30 w-10 h-16 sm:w-12 sm:h-20 transition-transform hover:scale-110 active:scale-95"
          >
            <svg viewBox="0 0 40 64" class="w-full h-full drop-shadow-md">
              <path d="M34 2 Q37 0 35 7 L29 27 Q28 31 29 35 L35 57 Q37 63 32 60 L3 35 Q0 32 3 29 Z" fill="#54caff" stroke="#62bce9" stroke-width="1.5" />
            </svg>
          </button>

          <!-- Envelope Grid Layout -->
          <div class="wii-message-pages grid w-full max-w-[760px] grid-cols-2 sm:grid-cols-3 items-center justify-items-center gap-x-6 gap-y-6 sm:gap-x-10 sm:gap-y-8">
            <button
              v-for="(msg, index) in visibleMessages"
              :key="msg.id"
              type="button"
              @click="openMessage(msg)"
              @mouseenter="playBlip(900)"
              class="wii-message-envelope relative flex aspect-[1.8/1] w-full max-w-[220px] flex-col items-center overflow-visible border border-gray-300/80 px-3 pt-5 pb-2 shadow-[0_5px_12px_rgba(55,65,81,0.18)] transition-transform hover:-translate-y-2 hover:shadow-[0_10px_18px_rgba(55,65,81,0.22)]"
              :class="[msg.color, index % 2 === 0 ? '-rotate-2' : 'rotate-2']"
            >
              <span class="wii-message-pin absolute -top-3 left-1/2 z-10 h-5 w-5 -translate-x-1/2 rounded-full"></span>
              <svg class="pointer-events-none absolute inset-0 h-full w-full" viewBox="0 0 210 118" preserveAspectRatio="none" aria-hidden="true">
                <path d="M1 2 105 68 209 2M1 116l76-54m132 54-76-54" fill="none" stroke="rgba(156,163,175,.24)" stroke-width="1.2" />
              </svg>
              <span class="relative z-[1] max-w-full truncate text-sm sm:text-base font-sans font-bold text-gray-700 tracking-wide mt-auto mb-2.5">{{ msg.sender }}</span>
            </button>
            <p v-if="messages.length === 0" class="col-span-full text-center font-sans text-lg text-gray-500 font-medium">No messages yet. Leave Azi a note!</p>
          </div>

          <!-- Right Navigation Arrow -->
          <button
            type="button"
            @click="changeMessagePage(1)"
            aria-label="Next messages"
            class="absolute right-4 sm:right-10 top-1/2 -translate-y-1/2 z-30 w-10 h-16 sm:w-12 sm:h-20 transition-transform hover:scale-110 active:scale-95"
          >
            <svg viewBox="0 0 40 64" class="w-full h-full drop-shadow-md">
              <path d="M6 2 Q3 0 5 7 L11 27 Q12 31 11 35 L5 57 Q3 63 8 60 L37 35 Q40 32 37 29 Z" fill="#54caff" stroke="#62bce9" stroke-width="1.5" />
            </svg>
          </button>
        </div>

        <!-- Read Note View -->
        <div v-if="msgView === 'read' && activeMessage" class="relative z-10 flex flex-grow flex-col items-center justify-center p-4 sm:p-8 pb-32 sm:pb-40">
          <div class="w-full max-w-xl overflow-hidden rounded-2xl border border-white/80 bg-white/95 shadow-[0_18px_45px_rgba(55,65,81,0.22)]">
            <div class="bg-gradient-to-r from-[#f97316] to-[#fb923c] px-4 py-3 text-center font-sans text-lg font-bold text-white shadow-sm">
              A note from {{ activeMessage.sender }}
            </div>
            <div class="wii-lined-paper min-h-[220px] p-6 sm:p-8 font-sans text-base sm:text-lg text-gray-700">
              <p class="whitespace-pre-wrap leading-[32px]">{{ activeMessage.content }}</p>
            </div>
            <div class="flex justify-between border-t border-gray-200 bg-gradient-to-b from-gray-50 to-gray-200 p-3 sm:p-4">
              <button type="button" @click="msgView = 'board'; playBlip(500)" class="rounded-full border-2 border-[#38bdf8] bg-gradient-to-b from-white to-[#e2e8f0] px-6 py-2 font-sans font-bold text-gray-600 shadow-sm hover:text-[#00a3e0]">Board</button>
              <button type="button" @click="msgView = 'compose'; playBlip(700)" class="rounded-full border-2 border-[#38bdf8] bg-gradient-to-b from-white to-[#e2e8f0] px-6 py-2 font-sans font-bold text-gray-600 shadow-sm hover:text-[#00a3e0]">Reply</button>
            </div>
          </div>
        </div>

        <!-- Write Note / Compose View -->
        <div v-if="msgView === 'compose'" class="relative z-10 flex flex-grow flex-col items-center justify-center p-4 sm:p-8 pb-32 sm:pb-40">
          <div class="w-full max-w-xl overflow-hidden rounded-2xl border border-white/80 bg-white/95 shadow-[0_18px_45px_rgba(55,65,81,0.22)]">
            <div class="bg-gradient-to-r from-[#f97316] to-[#fb923c] px-4 py-3 text-center font-sans text-lg font-bold text-white shadow-sm">
              Leave Azi a note
            </div>
            <div class="wii-lined-paper flex min-h-[220px] flex-col p-6 sm:p-8">
              <input
                v-model="composeName"
                type="text"
                placeholder="Your name"
                class="w-full bg-transparent font-sans text-lg outline-none text-gray-700 placeholder-gray-400 h-[32px] leading-[32px]"
              />
              <textarea
                v-model="composeContent"
                rows="6"
                placeholder="Write a short message..."
                class="w-full resize-none bg-transparent font-sans text-base sm:text-lg outline-none text-gray-700 placeholder-gray-400 leading-[32px] pt-[2px]"
              ></textarea>
            </div>
            <div class="flex justify-between border-t border-gray-200 bg-gradient-to-b from-gray-50 to-gray-200 p-3 sm:p-4">
              <button type="button" @click="msgView = 'board'; playBlip(500)" class="rounded-full border-2 border-[#38bdf8] bg-gradient-to-b from-white to-[#e2e8f0] px-6 py-2 font-sans font-bold text-gray-600 shadow-sm hover:text-[#00a3e0]">Cancel</button>
              <button type="button" @click="sendMessage" class="rounded-full border-2 border-[#38bdf8] bg-gradient-to-b from-white to-[#e2e8f0] px-6 py-2 font-sans font-bold text-gray-600 shadow-sm hover:text-[#00a3e0]">Send</button>
            </div>
          </div>
        </div>

        <!-- Message Board Footer -->
        <div class="wii-home-footer fixed bottom-0 left-0 right-0 h-[170px] sm:h-[195px] md:h-[220px] lg:h-[235px] z-30 pointer-events-none">
          <div class="relative w-full h-full flex items-end justify-between px-3 sm:px-6 md:px-16 pb-0 pointer-events-auto">
            <svg class="absolute inset-0 w-full h-full overflow-visible pointer-events-none z-0" viewBox="0 0 1000 235" preserveAspectRatio="none">
              <defs>
                <linearGradient id="wiiMsgFooterSide" x1="0" y1="0" x2="1" y2="1">
                  <stop offset="0" stop-color="#d1d5db" />
                  <stop offset="0.55" stop-color="#aeb6c0" />
                  <stop offset="1" stop-color="#c5cad1" />
                </linearGradient>
              </defs>
              <path d="M-50 0 C205 0 230 96 365 96 L635 96 C770 96 795 0 1050 0 L1050 235 L-50 235 Z" fill="url(#wiiMsgFooterSide)" opacity=".98" />
              <path d="M0 0 C210 0 230 95 365 95 L635 95 C770 95 790 0 1000 0" fill="none" stroke="#38bdf8" stroke-width="4" />
            </svg>

            <!-- LEFT: Write Note Button inside Grey Circular Background Plate -->
            <div class="footer-control relative z-10 flex items-center justify-center mb-4 sm:mb-6 md:mb-10">
              <div class="footer-button-plate footer-button-plate-left absolute -left-2 sm:-left-2 md:-left-3 w-[88px] h-[88px] sm:w-[120px] sm:h-[120px] md:w-[154px] md:h-[154px] bg-[#aeb6c0] rounded-full border border-white/55 z-0 pointer-events-none"></div>
              <div class="footer-button-shell overflow-hidden w-[68px] h-[68px] sm:w-[100px] sm:h-[100px] md:w-[132px] md:h-[132px] rounded-full bg-gradient-to-b from-[#f8fafc] to-[#cbd5e1] border-[4px] border-[#38bdf8] shadow-[0_5px_12px_rgba(55,65,81,0.18)] flex items-center justify-center relative z-10">
                <button
                  type="button"
                  @click="msgView = 'compose'; playBlip(700)"
                  aria-label="Write a note"
                  class="footer-round-button overflow-hidden w-full h-full rounded-full bg-gradient-to-b from-white to-[#e2e8f0] flex items-center justify-center text-[#718096] hover:text-[#00a3e0]"
                >
                  <svg class="w-10 h-10 md:w-14 md:h-14 fill-current" viewBox="0 0 24 24">
                    <path d="m3 17.25 11.06-11.06 3.75 3.75L6.75 21H3zm13.62-12.69 1.41-1.41a1.5 1.5 0 0 1 2.12 0l.7.7a1.5 1.5 0 0 1 0 2.12l-1.41 1.41z"/>
                  </svg>
                </button>
              </div>
            </div>

            <!-- CENTER: Date Display -->
            <div class="absolute left-1/2 top-0 -translate-x-1/2 z-20 w-full flex flex-col items-center pointer-events-none">
              <div class="absolute top-[110px] md:top-[112px] text-3xl md:text-[54px] font-bold text-[#6b7280] font-sans tracking-tight leading-none whitespace-nowrap">
                {{ currentDateBoard }}
              </div>
            </div>

            <!-- RIGHT: Wii Button inside Grey Circular Background Plate -->
            <div class="footer-control relative z-10 flex items-center justify-center mb-4 sm:mb-6 md:mb-10">
              <div class="footer-button-plate footer-button-plate-right absolute -right-2 sm:-right-2 md:-right-3 w-[88px] h-[88px] sm:w-[120px] sm:h-[120px] md:w-[154px] md:h-[154px] bg-[#aeb6c0] rounded-full border border-white/55 z-0 pointer-events-none"></div>
              <div class="footer-button-shell overflow-hidden w-[68px] h-[68px] sm:w-[100px] sm:h-[100px] md:w-[132px] md:h-[132px] rounded-full bg-gradient-to-b from-[#f8fafc] to-[#cbd5e1] border-[4px] border-[#38bdf8] shadow-[0_5px_12px_rgba(55,65,81,0.18)] flex items-center justify-center relative z-10">
                <button
                  type="button"
                  @click="activeChannelId = null; playBlip(400)"
                  aria-label="Return to Wii Menu"
                  class="footer-round-button w-full h-full rounded-full bg-gradient-to-b from-white to-[#e2e8f0] flex items-center justify-center text-[#718096] font-black text-3xl md:text-4xl hover:text-[#00a3e0]"
                >
                  Wii
                </button>
              </div>
            </div>

          </div>
        </div>

      </div>
    </transition>

  </div>
</template>

<style>
/* Custom Wii Cursor Styles */
.wii-cursor-container,
.wii-cursor-container * {
  cursor: auto;
}

@media (hover: hover) and (pointer: fine) {
  .wii-cursor-container,
  .wii-cursor-container * {
    cursor: none !important;
  }
}

.wii-custom-cursor {
  position: fixed;
  z-index: 2147483647;
  width: 49px;
  height: 71px;
  overflow: hidden;
  pointer-events: none;
  transform: translate(0, 0);
  transform-origin: top left;
  filter: drop-shadow(1px 2px 1px rgba(0, 0, 0, 0.35));
  transition: transform 100ms ease-out;
}

.wii-custom-cursor img {
  position: absolute;
  left: -72px;
  top: -47px;
  display: block;
  width: 194px;
  height: 162px;
  max-width: none;
}

.wii-custom-cursor--pressed {
  transform: scale(0.92);
}

.wii-home-footer {
  overflow: visible;
}

.wii-home-footer > div > svg {
  display: block;
  width: 100%;
  height: 100%;
}

.footer-button-shell {
  box-sizing: border-box;
  transition: border-color 0.2s ease, box-shadow 0.2s ease, transform 0.2s ease;
}

.footer-button-plate {
  top: 50%;
  transform: translateY(-50%);
  box-shadow: inset 0 4px 10px rgba(255, 255, 255, 0.28), inset 0 -6px 10px rgba(70, 78, 90, 0.16), 0 4px 8px rgba(70, 78, 90, 0.12);
}

.footer-button-plate-left {
  left: 50%;
  transform: translate(-50%, -50%);
}

.footer-button-plate-right {
  right: 50%;
  transform: translate(50%, -50%);
}

.footer-round-button {
  box-sizing: border-box;
  border: 0;
  outline: none;
  -webkit-tap-highlight-color: transparent;
  box-shadow: inset 0 2px 3px rgba(255, 255, 255, 0.9), inset 0 -2px 3px rgba(100, 116, 139, 0.06);
}

/* CRT Scanlines Overlay */
.crt-scanlines-topmost {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background: repeating-linear-gradient(
    180deg,
    rgba(0, 0, 0, 0.025) 0px,
    rgba(0, 0, 0, 0.025) 1px,
    transparent 1px,
    transparent 4px
  );
  z-index: 2147483647;
  pointer-events: none;
}

.wii-channel-shell {
  border-radius: 46px;
  background: transparent;
  isolation: isolate;
}

.wii-channel-display {
  background-position: center;
  background-size: cover;
  background-repeat: no-repeat;
  box-shadow: inset 0 2px 8px rgba(255, 255, 255, 0.18), inset 0 -8px 18px rgba(0, 0, 0, 0.12);
}

.wii-channel-glass {
  z-index: 1;
  background: linear-gradient(180deg, rgba(255, 255, 255, 0.16) 0%, rgba(255, 255, 255, 0.035) 18%, transparent 44%, rgba(255, 255, 255, 0.025) 78%, rgba(0, 0, 0, 0.08) 100%);
  border: 1px solid rgba(255, 255, 255, 0.22);
}

.wii-channel-footer {
  position: relative;
  z-index: 2;
  background-color: rgba(222, 226, 229, 0.94);
  background-image: repeating-linear-gradient(0deg, rgba(255, 255, 255, 0.5) 0, rgba(255, 255, 255, 0.5) 2px, rgba(150, 157, 164, 0.16) 2px, rgba(150, 157, 164, 0.16) 4px);
}

.wii-crt-channel {
  clip-path: none;
  border-radius: 22px !important;
  isolation: isolate;
}

.wii-channel-card {
  transition: transform 0.28s cubic-bezier(0.2, 0.8, 0.2, 1), box-shadow 0.28s ease;
}

.wii-channel-card:hover {
  transform: translateY(-5px) scale(1.03);
  box-shadow: 0 10px 22px rgba(0, 0, 0, 0.24);
}

.wii-channel-card:hover > div:nth-child(2) {
  animation: channel-icon-bounce 0.55s cubic-bezier(0.2, 0.8, 0.2, 1);
}

.wii-channel-card:active {
  transform: translateY(-1px) scale(0.98);
}

@keyframes channel-icon-bounce {
  0%, 100% {
    transform: translateY(0) scale(1);
  }
  35% {
    transform: translateY(-9px) scale(1.08);
  }
  65% {
    transform: translateY(2px) scale(0.98);
  }
}

@media (max-width: 639px) {
  .wii-home-footer > div > svg {
    transform: scaleY(0.9);
    transform-origin: bottom;
  }

  .wii-channel-card > svg,
  .wii-channel-card > div > svg {
    width: 3.5rem;
    height: 3.5rem;
  }

  .wii-channel-card .text-6xl {
    font-size: 2.75rem;
  }
}

@media (prefers-reduced-motion: reduce) {
  .wii-channel-card,
  .wii-channel-card:hover > div:nth-child(2) {
    transition: none;
    animation: none;
  }
}

.wii-empty-channel {
  background: repeating-linear-gradient(0deg, #cbd5e1, #cbd5e1 2px, #e2e8f0 2px, #e2e8f0 4px);
}

.digital-clock {
  font-family: 'Share Tech Mono', monospace;
}

.wii-lined-paper {
  background: repeating-linear-gradient(transparent, transparent 31px, #cbd5e1 31px, #cbd5e1 32px);
  line-height: 32px;
}

.wii-message-board {
  background:
    repeating-linear-gradient(180deg, rgba(0, 0, 0, 0.018) 0, rgba(0, 0, 0, 0.018) 1px, transparent 1px, transparent 4px),
    linear-gradient(180deg, #f8fafb 0%, #edf1f3 72%, #dfe4e8 100%);
}

.wii-message-envelope {
  background-image: linear-gradient(180deg, rgba(255, 255, 255, 0.36), transparent 45%);
}

.wii-message-pin {
  background: radial-gradient(circle at 32% 28%, #ffaaa0 0%, #ee5b50 42%, #bd2927 78%, #8f1719 100%);
  border: 1px solid rgba(130, 20, 20, 0.45);
  box-shadow: 0 2px 3px rgba(0, 0, 0, 0.24), inset 0 1px 1px rgba(255, 255, 255, 0.75);
}

.fade-enter-active, .fade-leave-active {
  transition: opacity 0.2s ease;
}
.fade-enter-from, .fade-leave-to {
  opacity: 0;
}
.fade-slow-enter-active, .fade-slow-leave-active {
  transition: opacity 1s ease;
}
.fade-slow-enter-from, .fade-slow-leave-to {
  opacity: 0;
}
</style>
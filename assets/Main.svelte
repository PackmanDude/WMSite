<script>
  const ip = 'play.whomine.net'
  const screenshots = [
    1, 2, 3, 4, 5, 6, 7, 8, 9, 10,
    11, 12, 13, 14, 15, 17, 18, 19,
    20, 22, 23, 24, 25, 26, 27, 28,
    30, 32, 33, 34, 35, 36, 37, 38,
    39, 40, 41, 42, 43, 44, 45, 46,
    47, 48, 49, 50, 51, 52, 53, 54,
    55, 56, 57, 58, 59, 61, 62, 63
  ]
  const ToastType = {
    INFO: 'info',
    SUCCESS: 'success',
    ERROR: 'error',
    WARNING: 'warning'
  }

  const toastTypeSet = new Set(Object.values(ToastType))
  const toastsContainer = document.querySelector('.toasts')
  const articles = document.querySelectorAll('article')
  const navItems = document.querySelector('#navItems')
  const navContent = document.querySelector('#navContent')
  const screenshot = document.querySelector('#screenshot-img')
  const randomButton = document.querySelector('#random-button')
  const headContainer = document.querySelector('.heads')
  const heads = [...headContainer.children]
  const personalInfoContainers = document.querySelectorAll('.personal-info > div')
  const bodyContainers = document.querySelectorAll('.skin > div')
  const teamContainer = document.querySelector('#team')
  const brightnessCache = {}

  let usedScreenshots = []
  let isHamburgering = false
  let isScreenshotLoading = false
  let currentIndex = 0
  let isUpdating = false
  let intervalId = null

  window.addEventListener('scroll', handleScroll)
  randomButton.addEventListener('click', getRandomScreenshot)
  headContainer.addEventListener('click', event => handleHeadClick(event.target))
  teamContainer.addEventListener('mouseenter', () => clearInterval(intervalId))
  teamContainer.addEventListener('mouseleave', startAutoScroll)

  document
  .querySelectorAll('a[href*="#"], button[onclick*="#"]')
  .forEach(element => element.addEventListener('click', handleAnchorClick))

  document
  .querySelectorAll('.ip-button')
  .forEach(element => element.addEventListener('click', handleIPButtonClick))

  document
  .querySelectorAll('.faq-master .item button')
  .forEach(element => element.addEventListener('click', handleFAQButtonClick))

  document
  .querySelector('#hamburger')
  .addEventListener('click', handleHamburgering)

  document
  .querySelector('#team-arrow-right')
  .addEventListener('click', () => handleTeamScroll(1))

  document
  .querySelector('#team-arrow-left')
  .addEventListener('click', () => handleTeamScroll(heads.length - 1))

  updateSelectedIndex()
  startAutoScroll()
  getRandomScreenshot()
  handleScroll()

  function updateSelectedIndex(previousIndex = 1) {
    if (isUpdating === true) return

    isUpdating = true
    const previousPersonalInfo = personalInfoContainers[previousIndex].classList
    const previousBody = bodyContainers[previousIndex].classList
    const currentPersonalInfo = personalInfoContainers[currentIndex].classList
    const currentBody = bodyContainers[currentIndex].classList

    heads[previousIndex].classList.remove('selected')

    previousPersonalInfo.add('fadeText-leave-to', 'fadeText-leave-active')
    setTimeout(() => {
      previousPersonalInfo.remove('selected', 'fadeText-leave-to', 'fadeText-leave-active')
    }, 350)

    previousBody.add('fadeImg-leave-to', 'fadeImg-leave-active')
    setTimeout(() => {
      previousBody.remove('selected', 'fadeImg-leave-to', 'fadeImg-leave-active')
    }, 350)

    heads[currentIndex].classList.add('selected')

    currentPersonalInfo.add('selected', 'fadeText-enter-from', 'fadeText-enter-active')
    setTimeout(() => {
      currentPersonalInfo.remove('fadeText-enter-from')
    }, 10)
    setTimeout(() => {
      currentPersonalInfo.remove('fadeText-enter-active')
      isUpdating = false
    }, 350)

    currentBody.add('selected', 'fadeImg-enter-from', 'fadeImg-enter-active')
    setTimeout(() => {
      currentBody.remove('fadeImg-enter-from')
    }, 10)
    setTimeout(() => {
      currentBody.remove('fadeImg-enter-active')
      isUpdating = false
    }, 350)
  }

  function startAutoScroll() {
    intervalId = setInterval(() => {
      if (
          isInViewport(teamContainer) === true
          && window.innerWidth >= 768
      ) {
        handleTeamScroll(1)
      }
    }, 5000)
  }

  function getRandomScreenshot() {
    if (isScreenshotLoading === true) return

    let unusedScreenshots = screenshots.filter(
        (_, index) => !usedScreenshots.includes(index)
    )

    if (unusedScreenshots.length === 0) {
      usedScreenshots = []
      unusedScreenshots = screenshots
    }

    const random = Math.trunc(Math.random() * unusedScreenshots.length)
    const index = screenshots.indexOf(unusedScreenshots[random])

    usedScreenshots.push(index)

    screenshot.style.opacity = '0'
    isScreenshotLoading = true

    setTimeout(() => {
      screenshot.src = `./assets/img/screenshots/${screenshots[index]}.webp`

      screenshot.onload = () => {
        randomButton.classList.toggle('dark', getCachedBrightness(screenshot))

        screenshot.style.opacity = '1'
        isScreenshotLoading = false
      }

      screenshot.onerror = () => {
        console.error(`Failed to load image: ${screenshot.src}`)
        isScreenshotLoading = false
      }
    }, 350)
  }

  function getCachedBrightness(image) {
    if (typeof brightnessCache[image.src] === 'undefined')
      brightnessCache[image.src] = calculateBrightness(image) > 128
    return brightnessCache[image.src]
  }

  function calculateBrightness(image) {
    const canvas = document.createElement('canvas')
    canvas.width = image.width
    canvas.height = image.height
    const context = canvas.getContext('2d')

    context.drawImage(image, 0, 0)

    const data = context.getImageData(0, 0, image.width, image.height).data
    let brightnessSum = 0

    for (let i = 0; i < data.length; i += 4) {
      brightnessSum += (data[i] + data[i + 1] + data[i + 2]) / 3
    }

    return brightnessSum * 4 / data.length
  }

  function isInViewport(
      element,
      threshold = 125
  ) {
    const {top, bottom, left, right} = element.getBoundingClientRect()
    const windowHeight = window.innerHeight ?? document.documentElement.clientHeight
    const windowWidth = window.innerWidth ?? document.documentElement.clientWidth

    return (
        top + threshold < windowHeight &&
        bottom - threshold > 0 &&
        left + threshold < windowWidth &&
        right - threshold > 0
    )
  }

  function scrollTo(target) {
    const windowHeight = window.innerHeight ?? document.documentElement.clientHeight
    const top = target.offsetTop - Math.max(0, windowHeight - target.offsetHeight) / 2

    window.scrollTo({
      top,
      behavior: 'smooth'
    })
  }

  function handleScroll() {
    for (const article of articles) {
      if (isInViewport(article)) article.classList.add('animate')
    }
  }

  function handleAnchorClick(event) {
    event.preventDefault()

    const target = document.querySelector(
        event.currentTarget.getAttribute('href')
        ?? event.currentTarget.getAttribute('onclick').split("'")[1]
    )

    if (target === null) {
      console.error(`Cannot resolve anchor ${event.currentTarget.getAttribute('href')} in file ${window.location.pathname}`)
    } else {
      scrollTo(target)
    }
  }

  async function handleIPButtonClick() {
    try {
      if ("clipboard" in navigator) {
        await navigator.clipboard.writeText(ip)
      } else {
        const textarea = document.createElement('textarea')
        textarea.value = ip

        document.body.appendChild(textarea)
        textarea.select()
        document.execCommand('copy')
        document.body.removeChild(textarea)
      }

      showToast('Айпи скопирован!', ToastType.SUCCESS)
    } catch (error) {
      showToast('Айпи не удалось скопировать :(', ToastType.ERROR)
      console.error('Failed to copy the IP: ', error)
    }
  }

  function handleFAQButtonClick() {
    const text = this.nextElementSibling
    text.style.display = ''

    setTimeout(() => {
      text.classList.toggle('open')
      this.lastElementChild.classList.toggle('open')
    }, 1)
    setTimeout(() => {
      text.style.display = text.classList.contains('open') ? '' : 'none'
    }, 500)
  }

  function handleHamburgering() {
    if (isHamburgering === true) return

    isHamburgering = true

    navItems.classList.toggle('open')
    navContent.classList.toggle('open')

    setTimeout(() => {
      navItems.style.display = navItems.classList.contains('open') ? 'flex' : ''
      isHamburgering = false
    }, 250)
  }

  function handleTeamScroll(offset) {
    openTeamMember((currentIndex + offset) % heads.length)
  }

  function handleHeadClick(head) {
    if (isUpdating === true) return

    const classList = head.classList

    if (
        classList.contains('head') === true
        && classList.contains('selected') === false
    ) openTeamMember(heads.indexOf(head))
  }

  function openTeamMember(index) {
    if (isUpdating === true) return

    const previousIndex = currentIndex
    currentIndex = index

    updateSelectedIndex(previousIndex)
  }

  function showToast(
      text,
      color = ToastType.INFO
  ) {
    if (typeof text !== 'string') {
      e('Toast text must be a string')
    }

    if (toastTypeSet.has(color) === false) {
      e('Toast color must be a valid ToastType')
    }

    const toast = document.createElement('div')

    toast.classList.add('toast', 'transitionIn', 'transitionOut', color)
    toast.textContent = text
    toastsContainer.appendChild(toast)

    setTimeout(() => {
      toastsContainer.removeChild(toast)
    }, 3500)
  }

  function e(o) { throw new Error(o) }
</script>

<nav id="navigation">
  <div id="navContent" class="content">
    <a class="logo" href="#header">
      <img alt="WhoMine Logo" src="assets/img/logo_rounded.webp" draggable="false">
    </a>
    <div id="hamburger" class="hamburger">
      <img src="assets/img/hamburger.svg" alt="Hamburger">
    </div>
    <ul id="navItems">
      <li>
        <a class="logo" href="#header">
          <img alt="WhoMine Logo" src="assets/img/logo_rounded.webp" draggable="false">
        </a>
      </li>
      <li class="m">
        <button onclick="location.href='#faq'" title="О Сервере" type="button">
          <span>О Сервере</span>
        </button>
      </li>
      <li class="m">
        <button onclick="location.href='https://docs.whomine.net'" title="Документация" type="button">
          <span>Документация</span>
          <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" style="margin-left: 0.5rem; margin-bottom: 0.05rem">
            <path stroke="#fff" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10 5H8.2c-1.1 0-1.7 0-2.1.2a2 2 0 0 0-.9.9C5 6.5 5 7 5 8.2v7.6c0 1.1 0 1.7.2 2.1.2.4.5.7.9.9.4.2 1 .2 2.1.2h7.6c1.1 0 1.7 0 2.1-.2.4-.2.7-.5.9-.9.2-.4.2-1 .2-2.1V14m1-5V4m0 0h-5m5 0-7 7"/>
          </svg>
        </button>
      </li>
      <li class="m">
        <button onclick="location.href='discord'" title="Discord" type="button">
          <span>Discord</span>
          <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" style="margin-left: 0.5rem; margin-bottom: 0.05rem">
            <path stroke="#fff" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10 5H8.2c-1.1 0-1.7 0-2.1.2a2 2 0 0 0-.9.9C5 6.5 5 7 5 8.2v7.6c0 1.1 0 1.7.2 2.1.2.4.5.7.9.9.4.2 1 .2 2.1.2h7.6c1.1 0 1.7 0 2.1-.2.4-.2.7-.5.9-.9.2-.4.2-1 .2-2.1V14m1-5V4m0 0h-5m5 0-7 7"/>
          </svg>
        </button>
      </li>
      <li class="m">
        <button class="ip-button" title="Скопировать" type="button">
          <img alt="Copy Button" src="assets/img/copy_button.svg" draggable="false">
          <span>play.whomine.net</span>
        </button>
      </li>
    </ul>
    <button class="ip-button" title="Скопировать" type="button">
      <img alt="Copy Button" src="assets/img/copy_button.svg" draggable="false">
      <span>play.whomine.net</span>
    </button>
  </div>
</nav>
<header id="header">
  <div class="page-bg">
    <div class="logo">
      <img src="assets/img/whomine290x290.webp" alt="whomine logo" draggable="false">
    </div>
  </div>
  <div class="scroll-down">
    <a href="#info" title="Спуститься ниже"></a>
  </div>
</header>
<main id="main">
  <div class="content">
    <article id="info" class="card-info info reverse">
      <img alt="T.A.R.D.I.S. Image" src="assets/img/tardis.webp" draggable="false">
      <div class="text">
        <h1>Кто мы?</h1>
        <p>WhoMine - это приватный RP-Survival сервер, где ты можешь стать кем угодно и воплотить свои фантазии в захватывающем игровом мире</p>
        <div class="ip-button">
          <img alt="Copy Button" src="assets/img/copy_button.svg" draggable="false">
          <span>play.whomine.net</span>
        </div>
      </div>
    </article>
    <article id="team" class="team">
      <div class="info">
        <div class="personal-info">
          <div id="personal-info-poloskun">
            <div class="about" style="color: #ea323c">
              <div class="name">Poloskun</div>
              <div class="job-title">
                <span style="color: white">Основатель</span>
              </div>
            </div>
            <p class="text">Привет, мой друг! Я отвечаю за техническую часть данного сервера и всегда готов помочь с любыми вопросами. Не стесняйся обращаться ко мне по любым вопросам, связанным с ошибками или идеями. Кроме того, я активно провожу время на сервере, где рад встрече с новыми игроками. Буду рад помочь и сделать наш сервер ещё лучше вместе с тобой!</p>
          </div>
          <div id="personal-info-demmicat">
            <div class="about" style="color: #ea323c">
              <div class="name">Demmicat</div>
              <div class="job-title">
                <span style="color: white">Основатель</span>
              </div>
            </div>
            <p class="text">На сегодняшний день я являюсь своеобразным «вышестоящим судом», который поможет вам в случае неразберихи. Я также помогаю серверу идейно и в качестве организатора ивентов. Стараюсь принимать участие в продвижении проекта, но теперь уже не как главное ответственное лицо, а как помощник для администрации.</p>
          </div>
          <div id="personal-info-sparrow">
            <div class="about" style="color: #f97216">
              <div class="name">Sparrow</div>
              <div class="job-title">
                <span style="color: white">Администратор</span>
              </div>
            </div>
            <p class="text">Привет! Я работаю с аудиторией сервера, принимаю их идеи о постройках, проектах и других аспектах развития. Готов помочь в развитии сервера, советами и поддержкой. Не забывай, что я также смогу помочь с различными видами неприятностей. Буду рад тебя видеть на нашем сервере, где ты сможешь насладиться увлекательным игровым опытом и встретить новых друзей!</p>
          </div>
        </div>
        <div class="head-list">
          <a id="team-arrow-left" class="arrow" style="transform: rotate(90deg)"></a>
          <div class="heads">
            <img id="poloskun" class="head" src="assets/img/team/poloskun_head.webp" alt="poloskun_" draggable="false">
            <img id="demmicat" class="head" src="assets/img/team/demmicat_head.webp" alt="demmicat" draggable="false">
            <img id="sparrow" class="head" src="assets/img/team/sparrow_head.webp" alt="sparrow" draggable="false">
          </div>
          <a id="team-arrow-right" class="arrow" style="transform: rotate(-90deg)"></a>
        </div>
      </div>
      <div class="skin">
        <div id="body-poloskun" style="background-image: url(assets/img/team/poloskun_body.webp)"></div>
        <div id="body-demmicat" style="background-image: url(assets/img/team/demmicat_body.webp)"></div>
        <div id="body-sparrow" style="background-image: url(assets/img/team/sparrow_body.webp)"></div>
      </div>
    </article>
    <article id="faq" class="faq">
      <div class="faq-master">
        <div class="item">
          <button type="button">
            <span>На какой версии сервер?</span>
            <img alt="Arrow" src="assets/img/down-arrow.svg" draggable="false">
          </button>
          <p class="text" style="display: none"><br>На данный момент сервер работает на версии <b>1.19.4</b></p>
        </div>
        <div class="item">
          <button type="button">
            <span>Нужна ли лицензия для входа?</span>
            <img alt="Arrow" src="assets/img/down-arrow.svg" draggable="false">
          </button>
          <p class="text" style="display: none"><br>Нет, лицензия не нужна, на сервере представлена система авторизации, которая позволяет вам защитить свой аккаунт</p>
        </div>
        <div class="item">
          <button type="button">
            <span>Как попасть на сервер?</span>
            <img alt="Arrow" src="assets/img/down-arrow.svg" draggable="false">
          </button>
          <p class="text" style="display: none"><br>Для того чтобы попасть на сервер, вам сперва необходимо зайти на наш дискорд сервер и написать администрации, после чего вам зададут несколько вопросов и вы сможете зайти на сервер</p>
        </div>
        <div class="item">
          <button type="button">
            <span>Какие моды используются на сервере?</span>
            <img alt="Arrow" src="assets/img/down-arrow.svg" draggable="false">
          </button>
          <p class="text" style="display: none"><br>Вы можете установить <b>PlasmoVoice</b>, чтобы использовать голосовой чат на сервере, а также <b>EmoteCraft</b> для использования эмоций в игре</p>
        </div>
        <div class="item">
          <button type="button">
            <span>Чем сервер уникален?</span>
            <img alt="Arrow" src="assets/img/down-arrow.svg" draggable="false">
          </button>
          <p class="text" style="display: none"><br>На сервере представлены уникальные механики, а также множество кастомных предметов, декораций и блоков. Подробнее об этих и других механиках вы можете узнать в нашей <a href="https://docs.whomine.net">документации</a></p>
        </div>
      </div>
      <a class="help" href="https://docs.whomine.net">
        <svg class="link-icon" xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none">
          <path stroke="#fff" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10 5H8.2c-1.1 0-1.7 0-2.1.2a2 2 0 0 0-.9.9C5 6.5 5 7 5 8.2v7.6c0 1.1 0 1.7.2 2.1.2.4.5.7.9.9.4.2 1 .2 2.1.2h7.6c1.1 0 1.7 0 2.1-.2.4-.2.7-.5.9-.9.2-.4.2-1 .2-2.1V14m1-5V4m0 0h-5m5 0-7 7"/>
        </svg>
        <div class="icon">
          <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 1000 1000" fill="none">
            <path fill="currentColor" fill-rule="evenodd" d="m562.168 159.724 325.95 162.727c15.062 7.519 15.298 28.898.404 36.746L465.19 582.283a82.875 82.875 0 0 1-75.639.83L123.74 450.409c-32.376-12.972-68.568 10.748-68.568 46.474 0 28.728 16.256 54.991 41.99 67.839l266.48 133.036c16.267-16.537 38.918-26.795 63.967-26.795 24.334 0 46.404 9.68 62.558 25.394L822.075 521.45a89.893 89.893 0 0 1-1.385-15.755c0-49.44 40.14-89.519 89.655-89.519S1000 456.255 1000 505.695c0 49.439-40.14 89.518-89.655 89.518-24.21 0-46.178-9.581-62.31-25.153L515.94 745.065a90.036 90.036 0 0 1 1.324 15.417c0 49.439-40.14 89.518-89.655 89.518s-89.655-40.079-89.655-89.518c0-4.572.343-9.063 1.006-13.451L68.622 612.068C26.566 591.072 0 548.153 0 501.205v-26.15c0-35.755 19.82-68.574 51.49-85.261l435.039-229.24a82.87 82.87 0 0 1 75.639-.83ZM427.609 794.912c19.044 0 34.483-15.415 34.483-34.43 0-19.016-15.439-34.431-34.483-34.431-19.044 0-34.482 15.415-34.482 34.431 0 19.015 15.438 34.43 34.482 34.43Zm517.219-289.217c0 19.015-15.438 34.43-34.483 34.43-19.044 0-34.482-15.415-34.482-34.43s15.438-34.43 34.482-34.43c19.045 0 34.483 15.415 34.483 34.43Z" />
          </svg>
        </div>
        <p class="title">Остались вопросы?</p>
        <p class="description">Узнай больше в нашей документации, где хранится вся полезная информация о сервере и его механиках</p>
      </a>
    </article>
    <article id="discord" class="card-info discord">
      <img src="assets/img/discord_pig.webp" alt="discord pig" draggable="false">
      <div class="text">
        <h1>Наш Дискорд</h1>
        <p>Мы думаем, что общение - это самое главное, по этому мы решили, что было бы круто сделать свой сервер в <b>Discord</b> где все люди могут общаться друг с другом. Заходите к нам, дабы окунутся в мир <b>#WhoMine</b>!</p>
        <button onclick="location.href='discord'" title="Перейти в дискорд" type="button">
          <span>Перейти в дискорд</span>
        </button>
      </div>
    </article>
    <article id="screen-gallery" class="screen-gallery">
      <div class="image-panel">
        <img id="screenshot-img" class="screenshot-img" alt="Screenshot" src="assets/img/screenshots/16.webp">
        <button id="random-button" class="button" type="button">
          <span>Случайный скриншот</span>
        </button>
      </div>
    </article>
  </div>
  <div class="toasts"></div>
</main>
<footer id="footer">
  <div class="content">
    <div class="author">
      <a class="logo" href="https://minersstudios.com">
        <img alt="MinersStudios Logo" src="https://raw.githubusercontent.com/MinersStudios/.github/main/assets/logos/logo_white.svg" draggable="false">
      </a>
      <p>WhoMine by <a class="transition-colors" href="https://minersstudios.com">MinersStudios</a></p>
    </div>
    <hr>
    <ul class="links">
      <li>
        <a class="transition-colors" href="telegram" title="Telegram канал">
          <svg xmlns="http://www.w3.org/2000/svg" fill="currentColor" viewBox="0 0 24 24">
            <path d="m2.5 10.5 18-6.8A1.2 1.2 0 0 1 22 5.1l-3.1 14a1.3 1.3 0 0 1-2 .8l-6.2-4.4a.8.8 0 0 1-.1-1.3l6.3-6v-.5a.3.3 0 0 0-.4 0L7.9 13a2 2 0 0 1-1.7.2L2.6 12a.8.8 0 0 1 0-1.5Z" />
          </svg>
        </a>
      </li>
      <li>
        <a class="transition-colors" href="discord" title="Discord сервер">
          <svg xmlns="http://www.w3.org/2000/svg" fill="currentColor" viewBox="0 0 24 24">
            <path d="M20.32 4.37a19.8 19.8 0 0 0-4.89-1.52.07.07 0 0 0-.08.04c-.2.38-.44.87-.6 1.25a18.27 18.27 0 0 0-5.5 0c-.16-.4-.4-.87-.6-1.25a.08.08 0 0 0-.09-.04 19.74 19.74 0 0 0-4.88 1.52.07.07 0 0 0-.04.03A20.26 20.26 0 0 0 .1 18.06a.08.08 0 0 0 .03.05 19.9 19.9 0 0 0 6 3.03.08.08 0 0 0 .08-.02c.46-.63.87-1.3 1.22-2a.08.08 0 0 0-.04-.1 13.1 13.1 0 0 1-1.87-.9.08.08 0 0 1 0-.12l.36-.3a.07.07 0 0 1 .08 0 14.2 14.2 0 0 0 12.06 0 .07.07 0 0 1 .08 0l.37.3a.08.08 0 0 1 0 .12 12.3 12.3 0 0 1-1.88.9.08.08 0 0 0-.04.1c.36.7.78 1.36 1.23 2a.08.08 0 0 0 .08.02 19.6 19.6 0 0 0 6-3.03.08.08 0 0 0 .04-.05c.5-5.18-.84-9.68-3.55-13.66a.06.06 0 0 0-.03-.03zM8.02 15.33c-1.18 0-2.16-1.08-2.16-2.42 0-1.33.96-2.42 2.16-2.42 1.21 0 2.18 1.1 2.16 2.42 0 1.34-.96 2.42-2.16 2.42zm7.97 0c-1.18 0-2.15-1.08-2.15-2.42 0-1.33.95-2.42 2.15-2.42 1.22 0 2.18 1.1 2.16 2.42 0 1.34-.94 2.42-2.16 2.42Z" />
          </svg>
        </a>
      </li>
      <li>
        <a class="transition-colors" href="github" title="GitHub">
          <svg xmlns="http://www.w3.org/2000/svg" fill="currentColor" viewBox="0 0 24 24">
            <path d="M12 0a12 12 0 0 0-3.8 23.4c.6.1.8-.3.8-.6v-2.2c-3.3.7-4-1.4-4-1.4-.6-1.4-1.4-1.8-1.4-1.8-1-.7.1-.7.1-.7 1.2 0 1.9 1.2 1.9 1.2 1 1.8 2.8 1.3 3.4 1 .2-.8.5-1.3.8-1.6-2.7-.3-5.5-1.3-5.5-6 0-1.2.5-2.3 1.3-3.1-.1-.4-.6-1.6.1-3.2 0 0 1-.3 3.3 1.2a11.5 11.5 0 0 1 6 0C17.3 4.7 18.3 5 18.3 5c.7 1.6.2 2.9.1 3.2.8.8 1.3 1.9 1.3 3.2 0 4.6-2.9 5.6-5.5 5.9.4.4.8 1.1.8 2.2v3.3c0 .3.2.7.8.6A12 12 0 0 0 12 0z" />
          </svg>
        </a>
      </li>
    </ul>
  </div>
  <p class="disclaimer">Not an official Minecraft product<br>We are in no way affiliated with or endorsed by <b>Mojang Synergies AB</b>, <b>Microsoft Corporation</b> or other rights-holders</p>
</footer>

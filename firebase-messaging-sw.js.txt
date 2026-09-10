// firebase-messaging-sw.js
//
// Precisa ficar publicado na MESMA PASTA que o index.html (ex: no seu caso,
// dentro da pasta "aura" do repositório, junto com o index.html — não numa
// subpasta separada). É esse arquivo que recebe a notificação push quando o
// app está FECHADO ou em segundo plano (quando o app está aberto, quem trata
// isso é o onForegroundMessage lá no index.html, e essa notificação "nativa"
// do sistema não aparece).

importScripts('https://www.gstatic.com/firebasejs/10.12.2/firebase-app-compat.js');
importScripts('https://www.gstatic.com/firebasejs/10.12.2/firebase-messaging-compat.js');

firebase.initializeApp({
  apiKey: "AIzaSyD4XkwODyk8bIfMPCOUdmOT5UIqdskCquc",
  authDomain: "aura-e8ef3.firebaseapp.com",
  projectId: "aura-e8ef3",
  storageBucket: "aura-e8ef3.firebasestorage.app",
  messagingSenderId: "208146657091",
  appId: "1:208146657091:web:0d1897f3eb0838010049f6",
});

const messaging = firebase.messaging();

messaging.onBackgroundMessage((payload) => {
  const title = (payload.notification && payload.notification.title) || 'Aura';
  const body = (payload.notification && payload.notification.body) || '';
  const options = {
    body,
    icon: '/icon-192.png', // troque pelo caminho real do ícone do app, se tiver um arquivo separado
    badge: '/icon-192.png',
    data: payload.data || {},
  };
  self.registration.showNotification(title, options);
});

// ao tocar na notificação, abre o app (ou foca a aba já aberta, se tiver uma)
self.addEventListener('notificationclick', (event) => {
  event.notification.close();
  // a Cloud Function já manda sempre um link completo (com o endereço do
  // site na frente) em event.notification.data.url — esse "./" aqui é só um
  // último recurso caso ele não venha por algum motivo, e funciona mesmo
  // quando o site está publicado numa subpasta (ex: GitHub Pages de projeto)
  const targetUrl = (event.notification.data && event.notification.data.url) || './';
  event.waitUntil(
    clients.matchAll({ type: 'window', includeUncontrolled: true }).then((windowClients) => {
      for (const client of windowClients) {
        if ('focus' in client) return client.focus();
      }
      if (clients.openWindow) return clients.openWindow(targetUrl);
    })
  );
});

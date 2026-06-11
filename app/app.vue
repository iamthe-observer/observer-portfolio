<template>
	<div
		class="page-container select-none"
		:class="{
			'work-state': isWorkState,
			'about-state': isAboutState,
			'has-perspective': hasPerspective,
		}"
	>
		<!-- Custom Cursor -->
		<div
			v-if="isDesktop"
			class="custom-cursor-dot"
			ref="cursorDot"
			:class="{ hovering: isHovering && !isOnLogo }"
		></div>
		<div
			v-if="isDesktop"
			class="custom-cursor-outline"
			ref="cursorOutline"
			:class="{ hovering: isHovering && !isOnLogo, 'on-logo': isOnLogo }"
		></div>

		<Transition name="bg-fade">
			<div
				v-if="activeClientIndex === null || !clients[activeClientIndex].vid"
				:key="'img-' + (activeClientIndex !== null ? activeClientIndex : 'default')"
				class="bg-image"
				:style="
					activeClientIndex !== null && clients[activeClientIndex].bg
						? { backgroundImage: `url(${clients[activeClientIndex].bg})` }
						: {}
				"
			></div>
			<video
				v-else
				:key="'vid-' + activeClientIndex"
				:src="clients[activeClientIndex].vid"
				class="bg-video"
				autoplay
				loop
				muted
				playsinline
			></video>
		</Transition>

		<!-- Close About button -->
		<button
			class="close-about-btn"
			v-show="isAboutState"
			@click.prevent="toggleAbout"
		>
			×
		</button>

		<main class="content-card" :class="{ 'is-contact': isContactState }">
			<!-- Floating Contact Close Button -->
			<Transition name="fade-delay">
				<button
					class="contact-close-btn"
					v-show="isContactState"
					@click.prevent="toggleContact"
				>
					×
				</button>
			</Transition>

			<!-- Floating Drag Indicator -->
			<div class="floating-drag-hint">
				<svg
					width="18"
					height="18"
					viewBox="0 0 24 24"
					fill="none"
					stroke="currentColor"
					stroke-width="2"
					stroke-linecap="round"
					stroke-linejoin="round"
				>
					<path d="M21 12H3"></path>
					<path d="M18 15l3-3-3-3"></path>
					<path d="M6 9l-3 3 3 3"></path>
				</svg>
			</div>

			<div class="card-front-content">
				<header class="card-header" v-show="!isAboutState">
					<div
						class="logo cursor-pointer"
						ref="logoRef"
						@click.prevent="transitionState('home')"
					>
						<span
							v-for="(letter, i) in logoLetters"
							:key="i"
							class="logo-letter"
							:class="{ revealed: revealedLetters[i] }"
							ref="letterRefs"
						>
							<span class="letter-default">{{ letter.from }}</span>
							<span class="letter-alt">{{ letter.to }}</span>
						</span>
					</div>
					<nav class="nav-links">
						<a href="#" class="nav-link" @click.prevent="toggleWork">WORK</a>
						<a href="#" class="nav-link" @click.prevent="toggleAbout">ABOUT</a>
					</nav>
					<button class="contact-btn header-contact" @click.prevent="toggleContact">
						CONTACT
					</button>
				</header>

				<div class="card-body">
					<!-- Home State Content -->
					<div class="home-content" v-show="!isWorkState && !isAboutState">
						<h1 class="main-heading">Branding, digital design<br />& development</h1>
						<p class="description">
							For small to medium businesses, startups,<br />
							and fellow professionals striving for growth<br />
							and new opportunities
						</p>
						<div class="action-buttons">
							<button class="portfolio-btn">PORTFOLIO</button>
						</div>
					</div>

					<!-- Work State Content -->
					<div class="work-content" v-show="isWorkState" style="display: none">
						<!-- Dynamic Info Panel above the carousel -->
						<Transition name="fade" mode="out-in">
							<div
								class="work-info-panel w-full grid grid-cols-1 md:grid-cols-12 gap-8 md:gap-6 text-left pt-0 pb-0 px-2"
								:key="activeClientIndex"
							>
								<!-- Col 1: Title & Year (Span 4) -->
								<div class="md:col-span-4 flex flex-col items-start pr-4 gap-4">
									<h2
										class="text-3xl md:text-[2.2rem] font-bold leading-[1.15] text-white/90 tracking-tight m-0"
										style="font-family: &quot;Playfair Display&quot;, serif"
									>
										{{ activeClient.title }}
									</h2>
									<span
										class="px-6 py-1.5 rounded-[20px] border border-white/20 text-[0.75rem] text-white/50 tracking-wide"
										>{{ activeClient.year }}</span
									>
								</div>

								<!-- Col 2: Challenge (Span 3) -->
								<div class="md:col-span-3 flex flex-col pt-1">
									<h3
										class="text-[0.75rem] text-white/40 mb-4 font-normal tracking-wide"
									>
										Challenge:
									</h3>
									<p class="text-[0.9rem] text-white/70 leading-[1.6] font-medium">
										{{ activeClient.challenge }}
									</p>
								</div>

								<!-- Col 3: Services (Span 2) -->
								<div class="md:col-span-2 flex flex-col pt-1">
									<h3
										class="text-[0.75rem] text-white/40 mb-4 font-normal tracking-wide"
									>
										Services:
									</h3>
									<div class="flex flex-wrap gap-2">
										<span
											v-for="service in activeClient.services"
											:key="service"
											class="px-3 py-1.5 rounded-[8px] border border-white/10 bg-white/5 text-[0.75rem] text-white/70 font-medium whitespace-nowrap"
											>{{ service }}</span
										>
									</div>
								</div>

								<!-- Col 4: Role (Span 3) -->
								<div class="md:col-span-3 flex flex-col pt-1">
									<h3
										class="text-[0.75rem] text-white/40 mb-4 font-normal tracking-wide"
									>
										Role:
									</h3>
									<p class="text-[0.9rem] text-white/70 leading-[1.6] font-medium">
										{{ activeClient.role }}
									</p>
								</div>
							</div>
						</Transition>

						<div
							class="carousel carousel-center rounded-box select-none cursor-grab active:cursor-grabbing w-full mt-4"
							ref="carouselRef"
							@mousedown="onMouseDown"
							@mouseleave="onMouseLeave"
							@mouseup="onMouseUp"
							@mousemove="onMouseMove"
							@scroll="onCarouselScroll"
						>
							<div
								v-for="(client, i) in clients"
								:key="i"
								class="carousel-item cursor-pointer"
								ref="carouselItems"
								@click="(e) => handleWorkClick(i, e)"
							>
								<div
									class="hover-3d work-card-3d"
									:class="{ 'work-card-active': activeClientIndex === i }"
								>
									<!-- content -->
									<figure class="work-card-figure">
										<img class="work-card-img" :src="client.bg" :alt="client.name" />
										<div class="work-card-overlay"></div>
										<div class="work-card-info">
											<span class="work-card-category">{{ client.category }}</span>
											<span class="work-card-name">{{ client.name }}</span>
										</div>
									</figure>
									<!-- 8 empty divs needed for the 3D effect -->
									<div></div>
									<div></div>
									<div></div>
									<div></div>
									<div></div>
									<div></div>
									<div></div>
									<div></div>
								</div>
							</div>
						</div>
					</div>

					<!-- About State Content -->
					<div class="about-content" v-show="isAboutState" style="display: none">
						<div class="about-image-wrapper">
							<div class="about-bg-image"></div>
							<h1 class="giant-text">OBSERVER</h1>
						</div>
						<div class="about-footer">
							<div class="about-name">RANSFORD LARBI</div>
							<div class="about-location">GHANA. WORKING WORLDWIDE</div>
						</div>
					</div>
				</div>
			</div>
			<!-- Contact Back Card -->
			<div class="card-back-content relative">
				<!-- Top Section -->
				<div
					class="flex justify-between items-start border-b border-white/10 pb-6 mb-6 w-full"
				>
					<div class="contact-titles text-left">
						<h2
							class="text-4xl mb-3 text-white tracking-tight font-medium"
							style="font-family: &quot;Playfair Display&quot;, serif"
						>
							Let's Talk
						</h2>
						<p
							class="text-[11px] text-gray-400 uppercase tracking-[0.2em] font-semibold"
						>
							Available for freelance opportunities
						</p>
					</div>
					<div class="flex items-start gap-4">
						<div
							class="qr-code w-16 h-16 bg-white/5 rounded-xl p-1.5 border border-white/10 flex items-center justify-center shrink-0"
						>
							<img src="/qr.svg" alt="QR Code" class="w-full h-full opacity-90" />
						</div>
					</div>
				</div>

				<!-- Middle Section: Contact Details Grid -->
				<div class="w-full h-full flex items-center justify-center">
					<div
						class="my-auto contact-links grid grid-cols-1 md:grid-cols-2 gap-y-16 gap-x-4 mt-2"
					>
						<a
							href="mailto:contact@observer.com"
							class="group flex items-center gap-4 text-sm hover:text-[#a855f7] transition-all duration-300"
						>
							<div
								class="w-10 h-10 rounded-full bg-white/5 border border-white/10 flex items-center justify-center group-hover:bg-[#a855f7]/10 group-hover:border-[#a855f7]/30 transition-all duration-300"
							>
								<svg
									class="w-4 h-4 text-gray-300 group-hover:text-[#a855f7] transition-colors"
									fill="none"
									stroke="currentColor"
									viewBox="0 0 24 24"
								>
									<path
										stroke-linecap="round"
										stroke-linejoin="round"
										stroke-width="1.5"
										d="M3 8l7.89 5.26a2 2 0 002.22 0L21 8M5 19h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v10a2 2 0 002 2z"
									></path>
								</svg>
							</div>
							<div class="flex flex-col">
								<span
									class="text-[10px] text-gray-500 uppercase tracking-wider font-semibold mb-1"
									>Email</span
								>
								<span class="font-light tracking-wide text-gray-200 text-xs"
									>topsquad3552@gmail.com</span
								>
							</div>
						</a>
						<a
							href="#"
							class="group flex items-center gap-4 text-sm hover:text-[#a855f7] transition-all duration-300"
						>
							<div
								class="w-10 h-10 aspect-square rounded-full bg-white/5 border border-white/10 flex items-center justify-center group-hover:bg-[#a855f7]/10 group-hover:border-[#a855f7]/30 transition-all duration-300"
							>
								<svg
									class="w-4 h-4 text-gray-300 group-hover:text-[#a855f7] transition-colors"
									fill="currentColor"
									viewBox="0 0 24 24"
								>
									<path
										d="M19 0h-14c-2.761 0-5 2.239-5 5v14c0 2.761 2.239 5 5 5h14c2.762 0 5-2.239 5-5v-14c0-2.761-2.238-5-5-5zm-11 19h-3v-11h3v11zm-1.5-12.268c-.966 0-1.75-.79-1.75-1.764s.784-1.764 1.75-1.764 1.75.79 1.75 1.764-.783 1.764-1.75 1.764zm13.5 12.268h-3v-5.604c0-3.368-4-3.113-4 0v5.604h-3v-11h3v1.765c1.396-2.586 7-2.777 7 2.476v6.759z"
									/>
								</svg>
							</div>
							<div class="flex flex-col">
								<span
									class="text-[10px] text-gray-500 uppercase tracking-wider font-semibold mb-1"
									>LinkedIn</span
								>
								<span class="font-light tracking-wide text-gray-200 text-xs"
									>linkedin.com/in/ransford-larbi-29b93422b</span
								>
							</div>
						</a>
						<a
							href="#"
							class="group flex items-center gap-4 text-sm hover:text-[#a855f7] transition-all duration-300"
						>
							<div
								class="w-10 h-10 rounded-full bg-white/5 border border-white/10 flex items-center justify-center group-hover:bg-[#a855f7]/10 group-hover:border-[#a855f7]/30 transition-all duration-300"
							>
								<svg
									class="w-4 h-4 text-gray-300 group-hover:text-[#a855f7] transition-colors"
									fill="currentColor"
									viewBox="0 0 24 24"
								>
									<path
										d="M11.944 0A12 12 0 0 0 0 12a12 12 0 0 0 12 12 12 12 0 0 0 12-12A12 12 0 0 0 12 0a12 12 0 0 0-.056 0zm4.962 7.224c.1-.002.321.023.465.14a.506.506 0 0 1 .171.325c.016.093.036.306.02.472-.18 1.898-.962 6.502-1.36 8.627-.168.9-.499 1.201-.82 1.23-.696.065-1.225-.46-1.9-.902-1.056-.693-1.653-1.124-2.678-1.8-1.185-.78-.417-1.21.258-1.91.177-.184 3.247-2.977 3.307-3.23.007-.032.014-.15-.056-.212s-.174-.041-.249-.024c-.106.024-1.793 1.14-5.061 3.345-.48.33-.913.49-1.302.48-.428-.008-1.252-.241-1.865-.44-.752-.245-1.349-.374-1.297-.789.027-.216.325-.437.892-.663 3.498-1.524 5.83-2.529 6.998-3.014 3.332-1.386 4.025-1.627 4.476-1.635z"
									/>
								</svg>
							</div>
							<div class="flex flex-col">
								<span
									class="text-[10px] text-gray-500 uppercase tracking-wider font-semibold mb-1"
									>Telegram</span
								>
								<span class="font-light tracking-wide text-gray-200 text-xs"
									>t.me/i_am_the_Observer</span
								>
							</div>
						</a>
						<a
							href="#"
							download
							class="group flex items-center gap-4 text-sm hover:text-[#a855f7] transition-all duration-300"
						>
							<div
								class="w-10 h-10 rounded-full bg-white/5 border border-white/10 flex items-center justify-center group-hover:bg-[#a855f7]/10 group-hover:border-[#a855f7]/30 transition-all duration-300"
							>
								<svg
									class="w-4 h-4 text-gray-300 group-hover:text-[#a855f7] transition-colors"
									fill="none"
									stroke="currentColor"
									viewBox="0 0 24 24"
								>
									<path
										stroke-linecap="round"
										stroke-linejoin="round"
										stroke-width="1.5"
										d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-4l-4 4m0 0l-4-4m4 4V4"
									></path>
								</svg>
							</div>
							<div class="flex flex-col">
								<span
									class="text-[10px] text-gray-500 uppercase tracking-wider font-semibold mb-1"
									>Resume</span
								>
								<span class="font-light tracking-wide text-gray-200 text-xs"
									>Download CV</span
								>
							</div>
						</a>
					</div>
				</div>
			</div>
		</main>
	</div>
</template>

<script setup>
import { ref, reactive, nextTick, onMounted, computed } from "vue";
import gsap from "gsap";

useSeoMeta({
	title: "Observer",
	description:
		"Observer is a creative developer specialized in WebGL, highly interactive experiences, and premium web design.",
	ogTitle: "Observer | Creative Developer & Designer",
	ogDescription:
		"Observer is a creative developer specialized in WebGL, highly interactive experiences, and premium web design.",
	ogImage: "/og-image.png",
	ogUrl: "https://observer.com",
	twitterTitle: "Observer | Creative Developer & Designer",
	twitterDescription:
		"Observer is a creative developer specialized in WebGL, highly interactive experiences, and premium web design.",
	twitterImage: "/og-image.png",
	twitterCard: "summary_large_image",
});

const clients = [
	{
		name: "EGdata",
		category: "Cloud Platform",
		bg: "",
		vid: "",
		title: "Ebbysgold Group Datapoint",
		year: "2026",
		challenge:
			"Develop a secure, centralized cloud datapoint system for the Ebbysgold Group to streamline enterprise analytics and operations.",
		services: ["Cloud Infrastructure", "Frontend Architecture", "UI/UX Design"],
		role:
			"Architected and built the robust frontend datapoint interface, ensuring fast, secure, and seamless access to critical group data.",
	},
	{
		name: "EG Dashboard",
		category: "Admin Dashboard",
		bg: "",
		vid: "",
		title: "Ebbysgold Travels Admin Dashboard",
		year: "2026",
		challenge:
			"Design and engineer a comprehensive internal management dashboard to handle high-volume travel bookings, customer data, and complex itineraries efficiently.",
		services: ["UI/UX Design", "Frontend Engineering", "Data Visualization"],
		role:
			"Developed the complete administrative interface with Nuxt, implementing complex data tables, real-time analytics, and secure Supabase integrations.",
	},
	{
		name: "Ebbysgold Travels",
		category: "Web App",
		bg: "/project_app2.png",
		vid: "",
		title: "Ebbysgold Travels and Tour",
		year: "2026",
		challenge:
			"Design a comprehensive and seamless travel booking experience that combines flight, hotel, and tour reservations into a unified, elegant platform.",
		services: ["UI/UX Design", "Web Development", "Brand Strategy"],
		role:
			"Designed the end-to-end interface and developed the frontend using Nuxt and Vue, ensuring an intuitive user flow from destination discovery to secure checkout.",
	},
	{
		name: "AdTonic",
		category: "Web App",
		bg: "/appproj1.png",
		vid: "/appproj1.mp4",
		title: "AdTonic Ad-Tech Platform",
		year: "2024",
		challenge:
			"Design a comprehensive dashboard for advertisers to track campaigns in real-time with granular analytics.",
		services: ["UI/UX Design", "Prototyping", "Design System"],
		role:
			"Led the interface design, transforming complex data tables into scannable, intuitive visual metrics.",
	},
	{
		name: "BlackVillage",
		category: "Branding",
		bg: "/appproj2.png",
		vid: "/appproj2.mp4",
		title: "BlackVillage Cultural Hub",
		year: "2023",
		challenge:
			"Develop a cohesive brand identity that honors heritage while projecting a modern, forward-thinking aesthetic.",
		services: ["Art Direction", "Brand Identity", "Typography"],
		role:
			"Created the primary logo, color palette, and digital brand guidelines from the ground up.",
	},
	{
		name: "Agami Dash",
		category: "Dashboard",
		bg: "/project_app2.png",
		vid: "",
		title: "Agami Analytics Dashboard",
		year: "2025",
		challenge:
			"Build a modular, customizable analytics dashboard for enterprise clients to visualize their growth.",
		services: ["UI Design", "Data Viz", "Component Library"],
		role:
			"Architected a flexible grid system and designed a suite of interactive charts and widgets.",
	},
	{
		name: "Agami Brand",
		category: "Identity",
		bg: "/project_app4.png",
		vid: "",
		title: "Agami Rebrand",
		year: "2025",
		challenge:
			"Refresh the legacy brand to align with their new pivot towards enterprise B2B software solutions.",
		services: ["Visual Strategy", "Logo Design", "Guidelines"],
		role:
			"Refined the brand mark and established a new, authoritative visual language for all corporate materials.",
	},
];

const carouselRef = ref(null);
const carouselItems = ref([]);

const activeClient = computed(
	() => clients[activeClientIndex.value] || clients[0],
);

const onCarouselScroll = () => {
	if (
		!carouselRef.value ||
		!carouselItems.value ||
		carouselItems.value.length === 0
	)
		return;

	const container = carouselRef.value;
	const containerCenter = container.scrollLeft + container.offsetWidth / 2;

	let closestIndex = 0;

	// Edge case bounds: if scrolled to the absolute start or end, force selection of the first/last item
	// This prevents the center-calculation from incorrectly highlighting the second/second-to-last item
	if (container.scrollLeft <= 5) {
		closestIndex = 0;
	} else if (Math.ceil(container.scrollLeft + container.offsetWidth) >= container.scrollWidth - 5) {
		closestIndex = carouselItems.value.length - 1;
	} else {
		let minDistance = Infinity;
		carouselItems.value.forEach((item, index) => {
			if (item) {
				const itemCenter = item.offsetLeft + item.offsetWidth / 2;
				const distance = Math.abs(itemCenter - containerCenter);
				if (distance < minDistance) {
					minDistance = distance;
					closestIndex = index;
				}
			}
		});
	}

	if (activeClientIndex.value !== closestIndex) {
		activeClientIndex.value = closestIndex;
	}
};

const logoRef = ref(null);
const letterRefs = ref([]);
const logoLetters = [
	{ from: "O", to: "R" },
	{ from: "B", to: "A" },
	{ from: "S", to: "N" },
	{ from: "E", to: "S" },
	{ from: "R", to: "F" },
	{ from: "V", to: "O" },
	{ from: "E", to: "R" },
	{ from: "R", to: "D" },
];
const revealedLetters = reactive([
	false,
	false,
	false,
	false,
	false,
	false,
	false,
	false,
]);

const isDesktop = ref(false);
const cursorDot = ref(null);
const cursorOutline = ref(null);
const isHovering = ref(false);
const isOnLogo = ref(false);

// Cursor outline radius (half of the 72px on-logo width)
const CURSOR_RADIUS = 36;

onMounted(() => {
	isDesktop.value = window.matchMedia("(pointer: fine)").matches;

	if (isDesktop.value) {
		nextTick(() => {
			gsap.set(cursorDot.value, { xPercent: -50, yPercent: -50 });
			gsap.set(cursorOutline.value, { xPercent: -50, yPercent: -50 });

			window.addEventListener("mousemove", (e) => {
				const posX = e.clientX;
				const posY = e.clientY;

				gsap.set(cursorDot.value, { x: posX, y: posY });
				gsap.to(cursorOutline.value, {
					x: posX,
					y: posY,
					duration: 0.15,
					ease: "power2.out",
				});

				const target = e.target;
				const isClickable = target.closest(
					"a, button, .cursor-pointer, .hover-3d, .nav-link, .carousel-item",
				);
				isHovering.value = !!isClickable;

				// Logo letter proximity detection
				const onLogo = target.closest(".logo");
				isOnLogo.value = !!onLogo;

				if (onLogo && letterRefs.value.length) {
					letterRefs.value.forEach((el, i) => {
						if (!el) return;
						const rect = el.getBoundingClientRect();
						const letterCenterX = rect.left + rect.width / 2;
						const letterCenterY = rect.top + rect.height / 2;
						const dist = Math.sqrt(
							Math.pow(posX - letterCenterX, 2) + Math.pow(posY - letterCenterY, 2),
						);
						revealedLetters[i] = dist < CURSOR_RADIUS;
					});
				} else {
					// Reset all letters when not on logo
					for (let i = 0; i < revealedLetters.length; i++) {
						revealedLetters[i] = false;
					}
				}
			});
		});
	}
});

const activeClientIndex = ref(null);
const isDragging = ref(false);
const startX = ref(0);
const scrollLeft = ref(0);
const hasDragged = ref(false);
const hasEverDragged = ref(false);

const onMouseDown = (e) => {
	isDragging.value = true;
	hasDragged.value = false;
	startX.value = e.pageX - carouselRef.value.offsetLeft;
	scrollLeft.value = carouselRef.value.scrollLeft;
};

const onMouseLeave = () => {
	isDragging.value = false;
};

const onMouseUp = () => {
	isDragging.value = false;
};

const onMouseMove = (e) => {
	if (!isDragging.value) return;
	e.preventDefault();
	const x = e.pageX - carouselRef.value.offsetLeft;
	const walk = (x - startX.value) * 2; // Scroll speed multiplier
	if (Math.abs(walk) > 5) {
		hasDragged.value = true;
		if (!hasEverDragged.value) {
			hasEverDragged.value = true;
			gsap.to(".floating-drag-hint", {
				opacity: 0,
				duration: 0.25,
				ease: "power2.out",
			});
		}
	}
	carouselRef.value.scrollLeft = scrollLeft.value - walk;
};

const handleWorkClick = (index, e) => {
	if (hasDragged.value) {
		e.preventDefault();
		e.stopPropagation();
		return;
	}
	activeClientIndex.value = index;

	// Automatically scroll the clicked card to the exact center of the carousel
	const container = carouselRef.value;
	const item = carouselItems.value[index];

	if (container && item) {
		const containerCenter = container.offsetWidth / 2;
		const itemCenter = item.offsetLeft + item.offsetWidth / 2;

		gsap.to(container, {
			scrollLeft: itemCenter - containerCenter,
			duration: 0.7,
			ease: "power3.out",
		});
	}
};

const isWorkState = ref(false);
const isAboutState = ref(false);
const isContactState = ref(false);
const hasPerspective = ref(false);

const wasWorkStateBeforeContact = ref(false);

const toggleContact = () => {
	isContactState.value = !isContactState.value;
	if (isContactState.value) {
		// Fade out drag hint immediately to prevent floating artifacts during 3D spin
		gsap.to(".floating-drag-hint", { opacity: 0, duration: 0.2 });

		// Remember and clear work state so card returns to neutral center
		wasWorkStateBeforeContact.value = isWorkState.value;
		if (isWorkState.value) {
			isWorkState.value = false;
		}
		hasPerspective.value = true;
	} else {
		// Restore work state if it was active before contact
		if (wasWorkStateBeforeContact.value) {
			isWorkState.value = true;
			wasWorkStateBeforeContact.value = false;
			if (!hasEverDragged.value) {
				gsap.to(".floating-drag-hint", {
					opacity: 1,
					duration: 0.5,
					delay: 0.6,
					ease: "power2.out",
				});
			}
		}
		setTimeout(() => {
			if (!isContactState.value) hasPerspective.value = false;
		}, 800);
	}
};

const transitionState = async (targetState) => {
	if (typeof window === "undefined") return;

	isContactState.value = false;
	hasPerspective.value = false;

	// Kill ongoing tweens to prevent conflicts
	gsap.killTweensOf([
		".home-content",
		".work-content",
		".about-content",
		".card-header",
	]);

	// Fade out current content
	let outTarget = ".home-content";
	if (isWorkState.value) {
		outTarget = ".work-content";
		if (!hasEverDragged.value) outTarget += ", .floating-drag-hint";
	}
	if (isAboutState.value) outTarget = ".about-content";

	// Hide card header if leaving home/work state and entering about
	if (targetState === "about") {
		outTarget += ", .card-header";
	}

	await gsap.to(outTarget, { opacity: 0, duration: 0.3, ease: "power2.out" });

	// Record if we were coming from About state
	const wasAbout = isAboutState.value;

	// Toggle states - CSS transitions will instantly begin smoothly scaling the card
	isWorkState.value = targetState === "work";
	isAboutState.value = targetState === "about";

	// Reset video if leaving work state
	if (targetState !== "work") {
		activeClientIndex.value = null;
	}

	// Wait for Vue to apply classes and v-show
	await nextTick();

	let inTarget = ".home-content";
	if (isWorkState.value) {
		inTarget = ".work-content";
		if (!hasEverDragged.value) inTarget += ", .floating-drag-hint";
	}
	if (isAboutState.value) inTarget = ".about-content";

	// If we came from about state and we are entering home or work, we need to fade in the header
	if (wasAbout && !isAboutState.value) {
		inTarget += ", .card-header";
	}

	// Show the container immediately so we can stagger its children
	gsap.set(inTarget, { opacity: 1 });

	let staggerElements = [];
	if (isWorkState.value) {
		staggerElements = Array.from(
			document.querySelectorAll(".work-info-panel > div, .carousel"),
		);
		if (!hasEverDragged.value) {
			const dragHint = document.querySelector(".floating-drag-hint");
			if (dragHint) staggerElements.push(dragHint);
		}
	} else if (isAboutState.value) {
		staggerElements = Array.from(document.querySelectorAll(".about-content > *"));
	} else {
		staggerElements = Array.from(document.querySelectorAll(".home-content > *"));
	}

	if (wasAbout && !isAboutState.value) {
		const header = document.querySelector(".card-header");
		if (header) staggerElements.unshift(header);
	}

	staggerElements = staggerElements.filter((el) => el);

	if (staggerElements.length > 0) {
		gsap.fromTo(
			staggerElements,
			{ opacity: 0, y: 15 },
			{
				opacity: 1,
				y: 0,
				duration: 0.6,
				delay: 0.35,
				stagger: 0.08,
				ease: "power2.out",
			},
		);
	} else {
		gsap.set(inTarget, { opacity: 0 });
		gsap.to(inTarget, {
			opacity: 1,
			duration: 0.5,
			delay: 0.4,
			ease: "power2.inOut",
		});
	}
};

const toggleWork = () => {
	transitionState(isWorkState.value ? "home" : "work");
};

const toggleAbout = () => {
	transitionState(isAboutState.value ? "home" : "about");
};
</script>

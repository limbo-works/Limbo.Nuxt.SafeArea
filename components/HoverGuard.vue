<template>
	<svg
		v-if="child"
		:key="`hover-guard-${id}`"
		:style="{
			position: 'absolute',
			width: `${svgWidth}px`,
			height: `${child.height}px`,
			pointerEvents: 'none',
			zIndex: 99999,
			top: `${
				direction === 'ttb' ? mousePosition.y - parent.y : child.y
			}px`,
			left: `${
				direction === 'ttb' ? child.x : mousePosition.x - parent.x
			}px`,
			// right: `${childX}px`,
		}"
	>
		<path
			v-if="showPath"
			pointer-events="auto"
			:stroke="[showBlocker ? 'red' : 'transparent']"
			:stroke-width="[showBlocker ? '1px' : 'none']"
			:fill="[showBlocker ? 'rgba(187,39,38,0.2)' : 'transparent']"
			:d="draw"
		></path>
	</svg>
</template>

<script setup>
const props = defineProps({
	parent: Object,
	child: Object,
	reverseDirection: Boolean,
	direction: {
		type: String,
		default: 'ltr',
	},
	showBlocker: {
		type: Boolean,
		default: false,
	},
});

const showPath = computed(() => {
	if (props.direction === 'ltr') {
		return !(mousePosition.value.x >= child.value.x) ? true : false;
	}
	if (props.direction === 'ttb') {
		return !(mousePosition.value.y > parent.value.y + parent.value.height)
			? true
			: false;
	}
});

const parent = ref({ x: 0, y: 0, height: 0, width: 0 });
const child = ref({ x: 0, y: 0, height: 0, width: 0 });

const mousePosition = ref({
	x: window.innerWidth,
	y: child.value.height,
});
const oldX = ref(0);
const oldY = ref(0);

const svgWidth = ref(0);
const svgHeight = ref(0);

const id = useId();

// Draw the path based on the menu rotation
const draw = computed(() => {
	if (props.direction === 'ltr') {
		return `M 0, ${mousePosition.value.y - child.value.y} L ${
			svgWidth.value
		}, ${svgHeight.value} L ${svgWidth.value}, ${child.value.y} z`;
	}
	if (props.direction === 'ttb') {
		return `M 0, ${svgHeight.value} L ${svgWidth.value}, ${
			svgHeight.value
		} L ${mousePosition.value.x - child.value.x}, ${child.value.x} z`;
	}
});

// Update the mouse position
const updateMousePosition = (e) => {
	let timeout;

	clearTimeout(timeout);

	if (props.direction === 'ltr') {
		if (e.clientX < oldX.value) {
			mousePosition.value.x = e.clientX + 5;
			mousePosition.value.y = e.clientY;
		}

		if (e.clientX < parent.value.x) {
			mousePosition.value.x = e.clientX + 5;
			mousePosition.value.y = e.clientY;
		}

		// check if the mouse has been stationary for a while
		timeout = setTimeout(() => {
			if (
				e.clientX === oldX.value &&
				e.clientY === oldY.value &&
				e.clientX > 0 &&
				e.clientY > 0
			) {
				mousePosition.value.x = e.clientX + 5;
				mousePosition.value.y = e.clientY;
			}
			updateSvgDimensions();
		}, 100);
	}
	if (props.direction === 'ttb') {
		if (e.clientY < oldY.value) {
			mousePosition.value.y = e.clientY + 5;
			mousePosition.value.x = e.clientX;
		}

		if (e.clientY < parent.value.y) {
			mousePosition.value.y = e.clientY + 5;
			mousePosition.value.x = e.clientX;
		}

		// check if the mouse has been stationary for a while
		timeout = setTimeout(() => {
			if (
				e.clientX === oldX.value &&
				e.clientY === oldY.value &&
				e.clientX > 0 &&
				e.clientY > 0
			) {
				mousePosition.value.y = e.clientY + 5;
				mousePosition.value.x = e.clientX;
			}
			updateSvgDimensions();
		}, 100);
	}

	oldX.value = e.clientX;
	oldY.value = e.clientY;
};

const getObjectDimensions = () => {
	parent.value = {
		x: parent.x,
		y: parent.y,
		width: parent.width,
		height: parent.height,
	} = props.parent?.getBoundingClientRect();

	child.value = {
		x: child.x,
		y: child.y,
		width: child.width,
		height: child.height,
	} = props.child?.getBoundingClientRect();
};

const updateSvgDimensions = () => {
	if (props.direction === 'ltr' || props.direction === 'rtl') {
		let gap = child.value.x - (parent.value.width + parent.value.x);
		svgWidth.value =
			parent.value.width + parent.value.x + gap - mousePosition.value.x;
		svgHeight.value = child.value.height;
	}
	if (props.direction === 'ttb' || props.direction === 'btt') {
		svgWidth.value = child.value.width;
		svgHeight.value =
			parent.value.height + parent.value.y - mousePosition.value.y;
	}
};

const addEventListeners = (e) => {
	e.addEventListener('mousemove', updateMousePosition);
	e.addEventListener('resize', getObjectDimensions);
	e.addEventListener('mousemove', getObjectDimensions);
	e.addEventListener('mousemove', updateSvgDimensions);
	e.addEventListener('resize', updateSvgDimensions);
};

const removeEventListeners = (e) => {
	e.removeEventListener('mousemove', updateMousePosition);
	e.removeEventListener('resize', getObjectDimensions);
	e.removeEventListener('mousemove', getObjectDimensions);
	e.removeEventListener('mousemove', updateSvgDimensions);
	e.removeEventListener('resize', updateSvgDimensions);
};

onUnmounted(() => {
	removeEventListeners(props.parent);
});

watch(
	() => props.parent,
	() => {
		addEventListeners(props.parent);
		getObjectDimensions();
		updateSvgDimensions();
	},
	{ immediate: true }
);
</script>

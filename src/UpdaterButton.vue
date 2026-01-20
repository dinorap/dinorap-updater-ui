<template>
  <button
    v-if="hasUpdate"
    @click="handleUpdate"
    class="dinorap-update-btn"
    :disabled="isUpdating"
    title="Nhấn để cập nhật phiên bản mới"
  >
    <span v-if="isUpdating" class="spinner">⏳</span>

    <span v-if="isUpdating"> Đang tải... App sẽ tự khởi động lại</span>
    <span v-else>⬇️ Cập nhật v{{ newVersion }}</span>
  </button>
</template>

<script setup>
import { ref, onMounted } from "vue";

// Nhận cấu hình từ bên ngoài truyền vào
const props = defineProps({
  // Địa chỉ backend (Ví dụ: http://localhost:8000)
  apiBase: {
    type: String,
    default: "http://localhost:8000",
    required: true,
  },
  // Prefix API (Mặc định khớp với backend SDK)
  apiPrefix: {
    type: String,
    default: "/api/update",
  },
});

const hasUpdate = ref(false);
const newVersion = ref("");
const isUpdating = ref(false);

const checkUpdate = async () => {
  try {
    // Gọi API: http://localhost:8000/api/update/check
    const url = `${props.apiBase}${props.apiPrefix}/check`;
    console.log("[UpdaterUI] Checking:", url);

    const res = await fetch(url);
    const data = await res.json();

    if (data.has_update) {
      hasUpdate.value = true;
      newVersion.value = data.version;
      console.log("[UpdaterUI] Found update:", data.version);
    }
  } catch (e) {
    // Lỗi thì im lặng, không làm phiền user, chỉ log console dev xem
    console.warn("[UpdaterUI] Check update failed (Backend might be offline):", e);
  }
};

const handleUpdate = async () => {
  if (
    !confirm(
      `Sếp có chắc chắn muốn cập nhật lên bản ${newVersion.value} không?\n\nỨng dụng sẽ tự động tắt và khởi động lại.`
    )
  ) {
    return;
  }

  isUpdating.value = true;
  try {
    const url = `${props.apiBase}${props.apiPrefix}/perform`;
    await fetch(url, { method: "POST" });

    // Backend đã nhận lệnh và sẽ tự restart app,
    // Frontend cứ treo thông báo đợi app tắt là vừa.
    alert("Hệ thống đang tải bản cập nhật. Vui lòng đợi trong giây lát...");
  } catch (e) {
    alert("Lỗi khi gửi lệnh cập nhật: " + e.message);
    isUpdating.value = false;
  }
};

// Check update ngay khi nút được mount vào giao diện
onMounted(() => {
  checkUpdate();
});
</script>

<style scoped>
/* Style nút bấm */
.dinorap-update-btn {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  background-color: #ef4444; /* Màu đỏ nổi bật */
  color: white;
  padding: 8px 16px;
  border-radius: 9999px; /* Bo tròn */
  font-weight: 700;
  font-size: 0.875rem;
  border: none;
  cursor: pointer;
  box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
  transition: all 0.2s;

  /* Animation nhấp nháy */
  animation: pulse-red 2s infinite;
}

.dinorap-update-btn:hover {
  background-color: #dc2626;
  transform: scale(1.05);
}

.dinorap-update-btn:disabled {
  background-color: #9ca3af; /* Màu xám khi đang tải */
  cursor: wait;
  animation: none; /* Tắt nhấp nháy khi đang tải */
  transform: none;
}

.spinner {
  animation: spin 1s linear infinite;
}

/* Định nghĩa hiệu ứng nhấp nháy */
@keyframes pulse-red {
  0% {
    transform: scale(0.95);
    box-shadow: 0 0 0 0 rgba(239, 68, 68, 0.7);
  }

  70% {
    transform: scale(1);
    box-shadow: 0 0 0 10px rgba(239, 68, 68, 0);
  }

  100% {
    transform: scale(0.95);
    box-shadow: 0 0 0 0 rgba(239, 68, 68, 0);
  }
}

@keyframes spin {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}
</style>

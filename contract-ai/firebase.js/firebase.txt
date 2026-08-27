import {
    collection,
    addDoc,
    serverTimestamp
} from "https://www.gstatic.com/firebasejs/11.0.0/firebase-firestore.js";

import {
    db
} from "./firebase.js";


// =========================
// 취득세 계산
// =========================

window.calculateTax = function () {

    const taxBase =
        Number(document.getElementById("taxBase").value);

    const taxRate =
        Number(document.getElementById("taxRate").value);

    if (!taxBase || !taxRate) {

        alert("과세표준과 세율을 입력하세요.");

        return;
    }

    const tax =
        taxBase * (taxRate / 100);

    document.getElementById("taxResult").innerHTML = `
        <strong>예상 취득세</strong><br><br>
        ${tax.toLocaleString()}원
    `;
};


// =========================
// 계약서 분석
// =========================

window.analyzeContract = async function () {

    const file =
        document.getElementById("contractFile").files[0];

    if (!file) {

        alert("계약서를 선택하세요.");

        return;
    }

    const result =
        document.getElementById("analysisResult");

    result.innerHTML =
        "계약서를 분석하고 있습니다...";

    const formData = new FormData();

    formData.append("file", file);

    try {

        const response =
            await fetch(
                "/.netlify/functions/analyze-contract",
                {
                    method: "POST",
                    body: formData
                }
            );

        const data =
            await response.json();

        if (!response.ok) {

            throw new Error(
                data.error || "분석 실패"
            );
        }

        result.innerHTML = `
            <h3>분석 결과</h3>
            <pre>${data.result}</pre>
        `;

    } catch (error) {

        result.innerHTML =
            `오류: ${error.message}`;
    }
};


// =========================
// AI 질문
// =========================

window.sendQuestion = async function () {

    const input =
        document.getElementById("userQuestion");

    const question =
        input.value.trim();

    if (!question) return;

    addMessage(question, "user");

    input.value = "";

    addMessage(
        "답변을 작성하고 있습니다...",
        "ai"
    );

    try {

        const response =
            await fetch(
                "/.netlify/functions/chat",
                {
                    method: "POST",

                    headers: {
                        "Content-Type":
                            "application/json"
                    },

                    body: JSON.stringify({
                        question
                    })
                }
            );

        const data =
            await response.json();

        if (!response.ok) {

            throw new Error(
                data.error || "AI 요청 실패"
            );
        }

        const messages =
            document.querySelectorAll(
                "#chatMessages .ai"
            );

        messages[messages.length - 1].innerHTML =
            data.answer;

    } catch (error) {

        const messages =
            document.querySelectorAll(
                "#chatMessages .ai"
            );

        messages[messages.length - 1].innerHTML =
            `오류: ${error.message}`;
    }
};


// =========================
// 채팅창 출력
// =========================

function addMessage(text, type) {

    const container =
        document.getElementById(
            "chatMessages"
        );

    const div =
        document.createElement("div");

    div.className =
        `message ${type}`;

    div.textContent = text;

    container.appendChild(div);

    container.scrollTop =
        container.scrollHeight;
}
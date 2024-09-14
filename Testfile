const axios = require('axios');
const fs = require('fs').promises;
const { HttpsProxyAgent } = require('https-proxy-agent');
const { Worker, isMainThread, parentPort, workerData } = require('worker_threads');
const colors = require('colors');

class HamsterKombatGame {
    constructor(config, accountIndex) {
        this.BASE_URL = 'https://api.hamsterkombatgame.io';
        this.TIMEOUT = 30000;
        this.UPGRADE_DIEUKIEN = config.UPGRADE_DIEUKIEN || 500000;
        this.promoCodeFile = 'code.txt';
        this.config = config;
        this.accountIndex = accountIndex;
        this.proxyIP = null;
        this.tokenFile = 'token.json';
    }

    async loadTokens() {
        try {
            const data = await fs.readFile(this.tokenFile, 'utf8');
            return JSON.parse(data);
        } catch (error) {
            if (error.code === 'ENOENT') {
                return {};
            }
            throw error;
        }
    }

    async saveToken(index, token) {
        const tokens = await this.loadTokens();
        tokens[index] = token;
        await fs.writeFile(this.tokenFile, JSON.stringify(tokens, null, 2));
    }

    async getToken(initDataRaw, retries = 5, backoffFactor = 0.5, timeout = 5000) {
        const tokens = await this.loadTokens();
        if (tokens[this.accountIndex]) {
            return tokens[this.accountIndex];
        }
    
        const url = 'https://api.hamsterkombatgame.io/auth/auth-by-telegram-webapp';
        const headers = {
            'Accept-Language': 'en-US,en;q=0.9',
            'Connection': 'keep-alive',
            'Origin': 'https://hamsterkombatgame.io',
            'Referer': 'https://hamsterkombatgame.io/',
            'Sec-Fetch-Dest': 'empty',
            'Sec-Fetch-Mode': 'cors',
            'Sec-Fetch-Site': 'same-site',
            'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/85.0.4183.102 Safari/537.36',
            'accept': 'application/json',
            'content-type': 'application/json'
        };
        const data = { initDataRaw };
    
        for (let attempt = 0; attempt < retries; attempt++) {
            try {
                const response = await axios.post(url, data, {
                    headers,
                    timeout
                });
                const token = response.data.authToken;
                await this.saveToken(this.accountIndex, token);
                return token;
            } catch (error) {
                await this.log(`Attempt ${attempt + 1} failed: ${error.message}`, 'warning');
                if (attempt < retries - 1) {
                    const delay = Math.pow(2, attempt) * backoffFactor * 1000;
                    await new Promise(resolve => setTimeout(resolve, delay));
                }
            }
        }
    
        await this.log("Failed to get token after multiple attempts.", 'error');
        return null;
    }

    async log(msg, type = 'info') {
        const timestamp = new Date().toLocaleTimeString();
        const accountPrefix = `[Tài khoản ${this.accountIndex + 1}]`;
        const ipPrefix = this.proxyIP ? `[${this.proxyIP}]` : '[Unknown IP]';
        let logMessage = '';
        
        switch(type) {
            case 'success':
                logMessage = `${accountPrefix}${ipPrefix} ${msg}`.green;
                break;
            case 'error':
                logMessage = `${accountPrefix}${ipPrefix} ${msg}`.red;
                break;
            case 'warning':
                logMessage = `${accountPrefix}${ipPrefix} ${msg}`.yellow;
                break;
            default:
                logMessage = `${accountPrefix}${ipPrefix} ${msg}`.blue;
        }
        
        console.log(logMessage);
        await this.randomDelay();
    }

    async randomDelay() {
        const delay = Math.floor(Math.random() * 1000) + 500; // Random delay between 500-1500ms
        await new Promise(resolve => setTimeout(resolve, delay));
    }

    createAxiosInstance(proxy) {
        const proxyAgent = new HttpsProxyAgent(proxy);
        return axios.create({
            baseURL: this.BASE_URL,
            timeout: this.TIMEOUT,
            headers: {
                'Content-Type': 'application/json'
            },
            httpsAgent: proxyAgent
        });
    }

    async checkProxyIP(proxy) {
        try {
            const proxyAgent = new HttpsProxyAgent(proxy);
            const response = await axios.get('https://api.ipify.org?format=json', {
                httpsAgent: proxyAgent 
            });
            if (response.status === 200) {
                this.proxyIP = response.data.ip;
                return this.proxyIP;
            } else {
                this.proxyIP = 'Unknown';
                return 'Unknown';
            }
        } catch (error) {
            await this.log(`Error khi kiểm tra IP của proxy: ${error}`, 'error');
            this.proxyIP = 'Error';
            return 'Error';
        }
    }

    async getBalanceCoins(dancay, authorization) {
        try {
            const response = await dancay.post('/clicker/sync', {}, {
                headers: {
                    'Authorization': `Bearer ${authorization}`
                }
            });

            if (response.status === 200) {
                return response.data.clickerUser.balanceCoins;
            } else {
                await this.log(`Không lấy được thông tin balanceCoins. Status code: ${response.status}`, 'error');
                return null;
            }
        } catch (error) {
            await this.log(`Error: ${error}`, 'error');
            return null;
        }
    }

    async selectExchange(dancay, authorization) {
        const exchanges = ['random', 'binance', 'okx'];
        const selectedExchange = exchanges[Math.floor(Math.random() * exchanges.length)];
        const payload = { exchangeId: selectedExchange };

        try {
            const response = await dancay.post('/clicker/select-exchange', payload, {
                headers: { 'Authorization': `Bearer ${authorization}` }
            });

            if (response.status === 200) {
                await this.log(`Successfully selected exchange: ${selectedExchange}`, 'success');
                return true;
            } else {
                await this.log(`Failed to select exchange. Status code: ${response.status}`, 'error');
                return false;
            }
        } catch (error) {
            await this.log(`Error selecting exchange: ${error}`, 'error');
            return false;
        }
    }

    async selectExchange(dancay, authorization) {
        const exchanges = ['random', 'binance', 'okx'];
        const selectedExchange = exchanges[Math.floor(Math.random() * exchanges.length)];
        const payload = { exchangeId: selectedExchange };

        try {
            const response = await dancay.post('/clicker/select-exchange', payload, {
                headers: { 'Authorization': `Bearer ${authorization}` }
            });

            if (response.status === 200) {
                await this.log(`Successfully selected exchange: ${selectedExchange}`, 'success');
                return true;
            } else {
                await this.log(`Failed to select exchange. Status code: ${response.status}`, 'error');
                return false;
            }
        } catch (error) {
            await this.log(`Error selecting exchange: ${error}`, 'error');
            return false;
        }
    }

    async buyUpgrades(dancay, authorization, specificUpgrade = null) {
        const DELAY_UPGRADE = this.config.DELAY_UPGRADE || false;
        const MIN_DELAY_UPGRADE = this.config.MIN_DELAY_UPGRADE || 0;
        const MAX_DELAY_UPGRADE = this.config.MAX_DELAY_UPGRADE || 1;

        try {
            const upgradesResponse = await dancay.post('/clicker/upgrades-for-buy', {}, {
                headers: { 'Authorization': `Bearer ${authorization}` }
            });

            if (upgradesResponse.status !== 200) {
                await this.log(`Không lấy được danh sách thẻ để nâng cấp: ${upgradesResponse.status}`, 'error');
                return false;
            }

            const upgrades = upgradesResponse.data.upgradesForBuy;
            let balanceCoins = await this.getBalanceCoins(dancay, authorization);
            let purchased = false;

            const upgradesToProcess = specificUpgrade ? [upgrades.find(u => u.id === specificUpgrade)] : upgrades;

            for (const upgrade of upgradesToProcess) {
                if (!upgrade) continue;

                if (upgrade.cooldownSeconds > 0) {
                    continue;
                }

                if (upgrade.isAvailable && !upgrade.isExpired && upgrade.price < this.UPGRADE_DIEUKIEN && upgrade.price <= balanceCoins) {
                    const buyUpgradePayload = {
                        upgradeId: upgrade.id,
                        timestamp: Math.floor(Date.now() / 1000)
                    };

                    try {
                        await this.log(`Thẻ ${upgrade.name} | Price: ${upgrade.price}`, 'info');
                        const response = await dancay.post('/clicker/buy-upgrade', buyUpgradePayload, {
                            headers: { 'Authorization': `Bearer ${authorization}` }
                        });

                        if (response.status === 200) {
                            await this.log(`(${Math.floor(balanceCoins)}) Nâng cấp thành công thẻ ${upgrade.name} lên Lv +${upgrade.level} | +${upgrade.profitPerHour}/h`, 'success');
                            purchased = true;
                            balanceCoins -= upgrade.price;

                            if (DELAY_UPGRADE) {
                                await new Promise(resolve => setTimeout(resolve, Math.floor(Math.random() * (MAX_DELAY_UPGRADE - MIN_DELAY_UPGRADE + 1) + MIN_DELAY_UPGRADE) * 1000));
                            } else {
                                await new Promise(resolve => setTimeout(resolve, 300));
                            }
                        }
                    } catch (error) {
                        const errorRes = error.response?.data;
                        const errorCode = errorRes?.error_code;

                        switch(errorCode) {
                            case 'INSUFFICIENT_FUNDS':
                                await this.log('Không đủ balance để nâng cấp', 'warning');
                                break;
                            case 'UPGRADE_COOLDOWN':
                                await this.log(`Thẻ đang trong thời gian đếm ngược ${errorRes.cooldownSeconds} giây.`, 'warning');
                                break;
                            case 'UPGRADE_MAX_LEVEL':
                                await this.log('Thẻ đã ở mức tối đa', 'info');
                                break;
                            case 'UPGRADE_NOT_AVAILABLE':
                                await this.log('Không đáp ứng đủ điều kiện mua thẻ', 'warning');
                                break;
                            case 'UPGRADE_HAS_EXPIRED':
                                await this.log('Thẻ đã hết hạn, bạn đến muộn', 'warning');
                                break;
                            case 'EXCHANGE_NOT_SELECTED':
                                await this.log('Exchange not selected. Attempting to select exchange...', 'warning');
                                const exchangeSelected = await this.selectExchange(dancay, authorization);
                                if (exchangeSelected) {
                                    await this.log('Exchange selected successfully. Retrying upgrade...', 'info');
                                    return this.buyUpgrades(dancay, authorization, specificUpgrade);
                                } else {
                                    await this.log('Failed to select exchange. Aborting upgrade.', 'error');
                                    return false;
                                }
                            default:
                                await this.log(errorRes, 'error');
                        }

                        if (specificUpgrade) return errorCode || 'error';
                    }
                }

                if (specificUpgrade) break;
            }

            if (specificUpgrade) {
                return purchased ? 'success' : 'not_purchased';
            } else {
                return purchased;
            }
        } catch (error) {
            await this.log('Lỗi không mong muốn, đang chuyển sang tài khoản tiếp theo', 'error');
            return specificUpgrade ? 'error' : false;
        }
    }

    async claimDailyCipher(dancay, authorization, cipher) {
        if (cipher) {
            try {
                const payload = {
                    cipher: cipher
                };
                const response = await dancay.post('/clicker/claim-daily-cipher', payload, {
                    headers: {
                        'Authorization': `Bearer ${authorization}`
                    }
                });

                if (response.status === 200) {
                    await this.log(`Đã giải mã morse ${cipher}`, 'success');
                } else {
                    await this.log(`Không claim được daily cipher. Status code: ${response.status}`, 'error');
                }
            } catch (error) {
                await this.log(`Đã giải mã morse!`, 'success');
            }
        }
    }

    async startAndClaimKeysMinigame(dancay, authorization) {
        try {
            const startResponse = await dancay.post('/clicker/start-keys-minigame', {}, {
                headers: {
                    'Authorization': `Bearer ${authorization}`
                }
            });

            if (startResponse.status === 200) {
                await this.log(`Đã bắt đầu keys minigame!`, 'success');
            } else {
                await this.log(`Không thể bắt đầu keys minigame. Status code: ${startResponse.status}`, 'error');
                return;
            }

            await this.randomDelay();

            const tokenSuffix = authorization.slice(-10);
            const randomPrefix = '0' + Math.floor(Math.random() * 1000000000).toString().padStart(9, '0');
            const cipher = `${randomPrefix}|${tokenSuffix}`;
            const base64Cipher = Buffer.from(cipher).toString('base64');

            const claimResponse = await dancay.post('/clicker/claim-daily-keys-minigame', { cipher: base64Cipher }, {
                headers: {
                    'Authorization': `Bearer ${authorization}`
                }
            });

            if (claimResponse.status === 200) {
                await this.log(`Đã claim daily keys minigame!`, 'success');
            } else {
                await this.log(`Không thể claim daily keys minigame. Status code: ${claimResponse.status}`, 'error');
            }
        } catch (error) {
            await this.log(`Đã claim daily keys minigame!`, 'success');
        }
    }

    async getPromoCodes() {
        try {
            const data = await fs.readFile(this.promoCodeFile, 'utf8');
            const lines = data.split('\n').map(line => line.trim()).filter(line => line !== '');
            const promoCodes = {
                ZOO: [], TRAIN: [], CUBE: [], MERGE: [], TWERK: [], POLY: [], TRIM: [], FLUF: [], STONE: []
            };
            lines.forEach(line => {
                const type = line.split('-')[0];
                if (promoCodes[type]) {
                    promoCodes[type].push(line);
                }
            });

            return promoCodes;
        } catch (error) {
            await this.log(`Error khi đọc mã khuyến mãi từ file: ${error}`, 'error');
            return {};
        }
    }

    async selectCodes(promoCodes) {
        const selectedCodes = {};
        const usedCodes = await this.loadUsedCodes();
    
        for (const type in promoCodes) {
            if (promoCodes[type].length > 0) {
                const codes = promoCodes[type].filter(code => !usedCodes.has(code));
                selectedCodes[type] = codes;
            }
        }
    
        return selectedCodes;
    }

    async checkPromoCodes(promoCodes, defaultRequiredCount = 4) {
        const validCodes = {};
        for (const [type, codes] of Object.entries(promoCodes)) {
            const requiredCount = type === 'FLUF' ? 8 : defaultRequiredCount;
            if (codes.length >= requiredCount) {
                validCodes[type] = codes;
            }
        }
        return validCodes;
    }

    async updatePromoCodeFile(promoCodes) {
        const allCodes = [].concat(...Object.values(promoCodes));
        await fs.writeFile(this.promoCodeFile, allCodes.join('\n'));
    }

    async loadUsedCodes() {
        try {
            const data = await fs.readFile('used_codes.txt', 'utf8');
            return new Set(data.split('\n').filter(code => code.trim() !== ''));
        } catch (error) {
            if (error.code === 'ENOENT') {
                return new Set();
            }
            throw error;
        }
    }
    
    async saveUsedCode(code) {
        await fs.appendFile('used_codes.txt', code + '\n');
    }

    async redeemPromoCodes(dancay, authorization, codes) {
        const successfulCodes = { ZOO: [], TRAIN: [], CUBE: [], MERGE: [], TWERK: [], POLY: [], TRIM: [], FLUF: [], STONE: [] };
        let totalSuccessful = 0;
        const maxTotalCodes = 8 * 4 + 8;
        
        for (const type of Object.keys(successfulCodes)) {
            if (!codes[type] || !Array.isArray(codes[type])) {
                continue; 
            }

            let count = 0;
            const maxCount = type === 'FLUF' ? 8 : 4;
            for (const code of codes[type]) {
                if (totalSuccessful >= maxTotalCodes || count >= maxCount) break; 
        
                try {
                    const payload = { promoCode: code };
                    const response = await dancay.post('/clicker/apply-promo', payload, {
                        headers: { 'Authorization': `Bearer ${authorization}` }
                    });
        
                    if (response.status === 200) {
                        await this.log(`Đã nhập mã khuyến mãi loại ${type} (${code}) thành công`, 'success');
                        successfulCodes[type].push(code);
                        await this.saveUsedCode(code);
                        totalSuccessful++;
                        count++;
                        
                        await this.removeSuccessfulCode(type, code);
                    } else {
                        if (response.status === 400) {
                            await this.removeInvalidCode(type, code);
                        }
                    }
                } catch (error) {                    
                    if (error.response && error.response.data && error.response.data.error_code === 'MaxKeysReceived') {
                        return;
                    }
                }
        
                await this.randomDelay();
            }
        }
        
        await this.log(`Đã nhập thành công ${totalSuccessful} mã khuyến mãi.`, 'success');
    }

    async removeInvalidCode(type, code) {
        try {
            const data = await fs.readFile(this.promoCodeFile, 'utf8');
            const lines = data.split('\n').map(line => line.trim()).filter(line => line !== '');
            const updatedLines = lines.filter(line => line !== code);
            await fs.writeFile(this.promoCodeFile, updatedLines.join('\n'));
            await this.log(`Đã xóa mã lỗi ${code} khỏi file`, 'info');
        } catch (error) {
            await this.log(`Error khi xóa mã lỗi khỏi file: ${error}`, 'error');
        }
    }

    async removeSuccessfulCode(type, code) {
        try {
            const data = await fs.readFile(this.promoCodeFile, 'utf8');
            const lines = data.split('\n').map(line => line.trim()).filter(line => line !== '');
            const updatedLines = lines.filter(line => line !== code);
            await fs.writeFile(this.promoCodeFile, updatedLines.join('\n'));
            await this.log(`Đã xóa mã đã sử dụng ${code} khỏi file`, 'info');
        } catch (error) {
            await this.log(`Error khi xóa mã đã sử dụng khỏi file: ${error}`, 'error');
        }
    }

    async getAvailableTaps(dancay, authorization) {
        try {
            const response = await dancay.post('/clicker/sync', {}, {
                headers: { 'Authorization': `Bearer ${authorization}` }
            });
            if (response.status === 200) {
                return response.data.clickerUser.availableTaps;
            } else {
                await this.log(`Không lấy được thông tin availableTaps. Status code: ${response.status}`, 'error');
                return null;
            }
        } catch (error) {
            await this.log(`Error khi lấy availableTaps: ${error}`, 'error');
            return null;
        }
    }

    async clickWithAPI(dancay, authorization, availableTaps) {
        try {
            const payload = {
                count: availableTaps,
                availableTaps: availableTaps,
                timestamp: Date.now()
            };
            const response = await dancay.post('/clicker/tap', payload, {
                headers: { 'Authorization': `Bearer ${authorization}` }
            });
            if (response.status === 200) {
                const data = response.data;
                const clickerUser = data.clickerUser;
                const requiredFields = {
                    Balance: Math.floor(clickerUser.balanceCoins),
                    Level: clickerUser.level,
                    availableTaps: clickerUser.availableTaps,
                    maxTaps: clickerUser.maxTaps
                };
                await this.log(`Tap Result: ${JSON.stringify(requiredFields)}`, 'success');
                return requiredFields;
            } else {
                await this.log(`Không bấm được. Status code: ${response.status}`, 'error');
            }
        } catch (error) {
            await this.log(`Error khi tap: ${error}`, 'error');
        }
        return null;
    }

    async checkAndBuyBoosts(dancay, authorization) {
        try {
            const boostsResponse = await dancay.post('/clicker/boosts-for-buy', {}, {
                headers: { 'Authorization': `Bearer ${authorization}` }
            });
            if (boostsResponse.status === 200 && boostsResponse.data.boostsForBuy) {
                const boosts = boostsResponse.data.boostsForBuy;
                const boostFullAvailableTaps = boosts.find(boost => boost.id === 'BoostFullAvailableTaps');
                if (boostFullAvailableTaps && boostFullAvailableTaps.cooldownSeconds === 0) {
                    const buyBoostPayload = {
                        boostId: 'BoostFullAvailableTaps',
                        timestamp: Math.floor(Date.now() / 1000)
                    };
                    await dancay.post('/clicker/buy-boost', buyBoostPayload, {
                        headers: { 'Authorization': `Bearer ${authorization}` }
                    });
                    await this.log(`Đã mua Full Energy cho token ${authorization.substring(0, 10)}...`, 'success');
                    return true;
                }
            } else {
                await this.log(`Không lấy được danh sách boosts. Status code: ${boostsResponse.status}`, 'error');
            }
        } catch (error) {
        }
        return false;
    }

    async performTapping(dancay, authorization) {
        while (true) {
            const availableTaps = await this.getAvailableTaps(dancay, authorization);
            if (availableTaps === null || availableTaps === 0) {
                await this.log("Không còn lượt tap.", 'info');
                break;
            }

            const tapResult = await this.clickWithAPI(dancay, authorization, availableTaps);
            if (tapResult === null) {
                await this.log("Lỗi khi thực hiện tap.", 'error');
                break;
            }

            const boughtBoost = await this.checkAndBuyBoosts(dancay, authorization);
            if (!boughtBoost) {
                break;
            }

            await this.randomDelay();
        }
    }

    async executeTasks(dancay, authorization) {
        try {
            const response = await dancay.post('/clicker/list-tasks', {}, {
                headers: { 'Authorization': `Bearer ${authorization}` }
            });
            if (response.status === 200) {
                const tasks = response.data.tasks;
                for (const task of tasks) {
                    if (!task.isCompleted) {
                        if (task.id === 'streak_days_special') {
                            await this.checkAndExecuteTask(dancay, authorization, 'streak_days_special', 'Đã điểm danh hàng ngày');
                        } else if (task.id !== 'invite_friends') {
                            await this.checkAndExecuteTask(dancay, authorization, task.id, `Làm nhiệm vụ ${task.id}`);
                        }
                    }
                }
            } else {
                await this.log(`Không lấy được danh sách nhiệm vụ. Status code: ${response.status}`, 'error');
            }
        } catch (error) {
            await this.log(`Lỗi khi xử lý nhiệm vụ: ${error}`, 'error');
        }
    }
    
    async checkAndExecuteTask(dancay, authorization, taskId, successMessage) {
        try {
            const checkResult = await dancay.post('/clicker/check-task', { taskId }, {
                headers: { 'Authorization': `Bearer ${authorization}` }
            });
            if (checkResult.data && checkResult.data.task.isCompleted) {
                await this.log(`${successMessage} thành công`, 'success');
            } else {
                await this.log(`${successMessage} thất bại`, 'error');
            }
        } catch (error) {
        }
    }

    async getComboCards() {
        const url = 'https://api21.datavibe.top/api/GetCombo';
        try {
            const response = await axios.post(url);
            const data = response.data;
            data.date = new Date().toLocaleDateString('en-GB');
            return data;
        } catch (error) {
            await this.log(`Failed getting Combo Cards. Error: ${error.message}`, 'error');
            return null;
        }
    }

    async executeCombo(dancay, authorization) {
        const comboData = await this.getComboCards();
        let comboPurchased = true;
        const MAXIMUM_PRICE_COMBO = this.config.MAXIMUM_PRICE_COMBO || 1000000;

        if (!comboData) {
            await this.log('Không thể lấy danh sách combo.', 'error');
            return;
        }

        let notReadyCombo = [];
        const dailyComboData = await this.claimDailyCombo(dancay, authorization);
        if (dailyComboData && dailyComboData.error_code) {
            if (dailyComboData.error_code === 'DAILY_COMBO_NOT_READY') {
                notReadyCombo = dailyComboData.error_message.split(':').pop().trim().split(',');
            } else if (dailyComboData.error_code === 'DAILY_COMBO_DOUBLE_CLAIMED') {
                return;
            }
        }

        const combo = comboData.combo || [];
        if (!combo.length) {
            await this.log('No combo data available.', 'warning');
            return;
        }

        const upgrades = await this.availableUpgrades(dancay, authorization);
        for (const comboItem of combo) {
            if (notReadyCombo.includes(comboItem)) {
                await this.log(`Has executed ${comboItem}`, 'info');
                continue;
            }

            const upgradeDetails = upgrades.find(u => u.id === comboItem);
            const upgradePriceDict = upgrades.find(u => u.price);
            if (upgradePriceDict) {
                const upgradePrice = upgradePriceDict.price;
                if (upgradePrice > MAXIMUM_PRICE_COMBO) {
                    await this.log('Price combo is over max price', 'warning');
                    return;
                }
            }
            if (!upgradeDetails) {
                await this.log(`Failed to find details ${comboItem}`, 'error');
                continue;
            }

            const status = await this.buyUpgrades(dancay, authorization, comboItem);
            if (status === 'success') {
                await new Promise(resolve => setTimeout(resolve, 1000));
            } else {
                comboPurchased = false;
                await new Promise(resolve => setTimeout(resolve, 1000));
                break;
            }
        }
        if (comboPurchased) {
            await this.claimDailyCombo(dancay, authorization);
        } else {
            await this.log('Combo not fully purchased', 'warning');
        }
    }

    async claimDailyCombo(dancay, authorization) {
        const url = 'https://api.hamsterkombatgame.io/clicker/claim-daily-combo';
        try {
            const res = await dancay.post(url, {}, {
                headers: { 'Authorization': `Bearer ${authorization}` }
            });
            const data = res.data;
            const bonusCoins = data.dailyCombo?.bonusCoins || 0;
            await this.log(`Daily combo reward +${bonusCoins}`, 'success');
            return data;
        } catch (error) {
            const errorRes = error.response?.data;
            const errorCode = errorRes?.error_code;
            if (errorCode === 'DAILY_COMBO_NOT_READY') {
                await this.log('Daily combo not ready.', 'info');
            } else if (errorCode === 'DAILY_COMBO_DOUBLE_CLAIMED') {
                await this.log('Combo has already been claimed', 'info');
            } else {
                await this.log(`Failed to claim daily combo: ${errorRes}`, 'error');
            }
            return errorRes;
        }
    }

    async availableUpgrades(dancay, authorization) {
        try {
            const res = await dancay.post('/clicker/upgrades-for-buy', {}, {
                headers: { 'Authorization': `Bearer ${authorization}` }
            });
            return res.data.upgradesForBuy;
        } catch (error) {
            await this.log(`Failed to get upgrade list: ${error.message}`, 'error');
            return [];
        }
    }

    async runForToken(initDataRaw, proxy) {
        const authorization = await this.getToken(initDataRaw);
        if (!authorization) {
            await this.log(`Không thể lấy token cho tài khoản ${this.accountIndex}`, 'error');
            return;
        }

        const ip = await this.checkProxyIP(proxy);
        await this.log(`Bắt đầu xủ lý tài khoản ${authorization.substring(0, 10)}...`, 'info');
        
        const dancay = this.createAxiosInstance(proxy);
        await this.executeTasks(dancay, authorization);
        await this.performTapping(dancay, authorization);
        await this.claimDailyCipher(dancay, authorization, this.config.cipher);
        await this.startAndClaimKeysMinigame(dancay, authorization);
        
        if (this.config.promo) {
            try {
                const promoCodes = await this.getPromoCodes();
                const validCodes = await this.checkPromoCodes(promoCodes);
                if (Object.keys(validCodes).length > 0) {
                    const selectedCodes = await this.selectCodes(validCodes);
                    await this.redeemPromoCodes(dancay, authorization, selectedCodes);
                } else {
                    await this.log('Không có mã khuyến mãi nào đủ số lượng để nhập.', 'warning');
                }
            } catch (error) {
                await this.log(`Error khi xử lý mã khuyến mãi: ${error}`, 'error');
            }
        }

        if (this.config.shouldUpgrade) {
            while (true) {
                const success = await this.buyUpgrades(dancay, authorization);
                if (!success) {
                    break;
                }
            }
        }

        await this.executeCombo(dancay, authorization);
    }
}

async function workerThread(workerData) {
    const { initDataRaw, proxy, config, accountIndex } = workerData;
    const game = new HamsterKombatGame(config, accountIndex);

    await game.runForToken(initDataRaw, proxy);
    parentPort.postMessage('done');
}

async function main() {
    const initDataRawList = await fs.readFile('data.txt', 'utf8')
        .then(data => data.split('\n').map(line => line.trim()).filter(line => line !== ''));
    const proxyList = await fs.readFile('proxy.txt', 'utf8')
        .then(data => data.split('\n').map(line => line.trim()).filter(line => line !== ''));
    const config = JSON.parse(await fs.readFile('config.json', 'utf8'));

    const THREADS = config.threads || 10;

    async function runWorker(initDataRaw, proxy, accountIndex) {
        return new Promise((resolve, reject) => {
            const worker = new Worker(__filename, {
                workerData: { initDataRaw, proxy, config, accountIndex }
            });

            worker.on('message', resolve);
            worker.on('error', reject);
            worker.on('exit', (code) => {
                if (code !== 0) reject(new Error(`Luồng bị dừng với code ${code}`));
            });
        });
    }

    async function processAccounts() {
        let currentIndex = 0;

        while (currentIndex < initDataRawList.length) {
            const workerPromises = [];

            for (let i = 0; i < THREADS && currentIndex < initDataRawList.length; i++) {
                const initDataRaw = initDataRawList[currentIndex];
                const proxy = proxyList[currentIndex % proxyList.length];
                
                workerPromises.push(runWorker(initDataRaw, proxy, currentIndex));
                currentIndex++;
            }

            await Promise.all(workerPromises);
            await new Promise(resolve => setTimeout(resolve, 3000));
        }

        console.log('Đã xử lý hết tất cả tài khoản, chờ 1 chút để chạy lại...');
        await new Promise(resolve => setTimeout(resolve, 60000)); 
    }

    while (true) {
        await processAccounts();
    }
}

if (isMainThread) {
    main().catch(console.error);
} else {
    workerThread(workerData);
}

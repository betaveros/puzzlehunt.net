---
layout: page
short_title: Checker
title: Client-Side Answer Checker
---
Answers are considered as a string of letters A–Z by changing lowercase letters to uppercase and stripping any other characters. Uses [scrypt-js](https://github.com/ricmoo/scrypt-js). [What is this?](/checker/about)

<p id="loading">Loading...</p>

<form id="check">
<h3 id="check-label">Check your answer:</h3>
<p><input type="text" id="check-input" name="check-input" /> <button type="submit" id="check-submit">Check</button></p>
<p id="check-out"></p>
<p id="check-solution"></p>

<p>You can also <a href="#">create your own checker</a>.</p>
</form>

<form id="gen">
<h3>Create an answer checker:</h3>
<p>Enter your answer below and click Generate URL:</p>
<p>Correct answer: <input type="text" id="gen-input" /></p>
<p>What this answer checker is for (optional): <input type="text" id="gen-label" /></p>
<label for="gen-solution">Text to reveal after a correct answer (optional):</label>
<p><textarea id="gen-solution"></textarea></p>
<button type="submit">Generate URL</button>
<p id="gen-outer"><span id="gen-out"></span><a id="gen-link"></a></p>
</form>

<script type="text/javascript" src="/js/scrypt.js"></script>

<script type="text/javascript">
const encoder = new TextEncoder();
const decoder = new TextDecoder();
function b64OfArray(arr) {
	const carr = [];
	arr.forEach((u8) => {
		carr.push(String.fromCharCode(u8));
	});
	return btoa(carr.join(""));
}
function unb64(s) {
	const bs = atob(s);
	const uarr = new Uint8Array(bs.length);
	for (let i = 0; i < bs.length; i++) {
		uarr[i] = bs.charCodeAt(i);
	}
	return uarr;
}

function canonicalize(rawGuess) {
	rawGuess = rawGuess.toUpperCase();
	let canon = "";
	for (var i = 0; i < rawGuess.length; i++) {
		if (/[A-Z]/.test(rawGuess[i])) {
			canon += rawGuess[i];
		}
	}
	return canon;
}

// These security parameters are weaker than what most settings would need, to
// keep things reasonably fast, since our pure JavaScript library is slower
// than other options, and URLs reasonably compact. A brute-forcer who ran
// scrypt elsewhere with these parameters could go much faster, but this
// setting really isn't high-stakes.

// Don't copy my parameters into "actual crypto code" (why would you do that.
// just. why)
function generateLocalSalt() {
	let saltArr = new Uint8Array(12);
	if (window.crypto && window.crypto.getRandomValues) {
		window.crypto.getRandomValues(saltArr);
	} else {
		// Not secure, but like I said, I think cryptographic guarantees just
		// aren't worth breaking over.
		for (let i = 0; i < saltArr.length; i++) {
			saltArr[i] = Math.floor(Math.random()*256);
		}
	}
	return b64OfArray(saltArr);
}

function generateHash(label, answer, solution, callback) {
	const version = '1';
	// The caller should canonicalize the answer!
	const salt = generateLocalSalt();

	let ciphertextPromise = null;
	if (solution === "") {
		ciphertextPromise = new Promise(resolve => resolve(null));
	} else {
		// "-encrypt" for domain separation
		const encryptionSalt = encoder.encode("puzzlehunt.net/checker#" + version + '-encrypt#' + salt + '#' + label);
		let encodedSolution = encoder.encode(solution);
		// very lazy xor cipher
		ciphertextPromise = scrypt.scrypt(encoder.encode(answer), encryptionSalt, 4096, 8, 1, encodedSolution.length, function (progress) {
			callback({ 'progress': progress, 'encrypting': true });
		}).then(function (key) {
			for (let i = 0; i < encodedSolution.length; i++) {
				encodedSolution[i] ^= key[i];
			}
			return encodedSolution;
		});
	}
	ciphertextPromise.then(function (ciphertext) {
		const encodedCiphertext = ciphertext ? b64OfArray(ciphertext) : null;
		// Note: add the label even if it's empty. Also assume the label is ASCII
		// (by being v0 URI-encoded or v1 base64ed) already.
		const fullSalt = encoder.encode("puzzlehunt.net/checker#" + version + '#' + salt + '#' + label
			+ (encodedCiphertext ? "#" + encodedCiphertext : ""));
		// N = 4096 = 2^12 = memory cost factor (a normal lower bound,
		// e.g. libsodium's crypto_pwhash_scryptsalsa208sha256_OPSLIMIT_MIN, is
		// 2^15; so you can see how skimpy on computation we are. It's because
		// we're using a pure-JavaScript implementation, which automatically makes
		// the constant factors huge.)
		// r = 8 = block size factor
		// p = 1 = parallelization parameter
		// dkLen = 24 = desired key length
		scrypt.scrypt(encoder.encode(answer), fullSalt, 4096, 8, 1, 24, function (progress) {
			callback({ 'progress': progress });
		}).then(function (key) {
			let ret = {
				'version': version,
				'salt': salt,
				'hash': b64OfArray(key),
			};
			if (encodedCiphertext) {
				ret.ciphertext = encodedCiphertext;
			}
			callback(ret);
		});
	});
}

function checkHash(version, label, salt, hash, answer, ciphertext, callback) {
	// Weirdly, the version doesn't yet affect this part of the code.
	if (version !== '0' && version !== '1') {
		callback({
			'error': 'Unsupported version: ' + version,
		});
	}
	// The caller should canonicalize the answer!
	// Note: add the label even if it's empty. Also assume the label is ASCII
	// (by being v0 URI-encoded or v1 base64ed) already.
	const fullSalt = encoder.encode("puzzlehunt.net/checker#" + version + '#' + salt + '#' + label
		+ (ciphertext ? "#" + ciphertext : ""));
	scrypt.scrypt(encoder.encode(answer), fullSalt, 4096, 8, 1, 24, function (progress) {
		callback({ 'progress': progress });
	}).then(function (key) {
		if (b64OfArray(key) === hash) {
			if (ciphertext === "") {
				callback({ 'correct': true });
			} else {
				const encryptionSalt = encoder.encode("puzzlehunt.net/checker#" + version + '-encrypt#' + salt + '#' + label);
				const encodedSolution = unb64(ciphertext);
				scrypt.scrypt(encoder.encode(answer), encryptionSalt, 4096, 8, 1, encodedSolution.length, function (progress) {
					callback({ 'correct': true, 'progress': progress });
				}).then(function (key) {
					for (let i = 0; i < encodedSolution.length; i++) {
						encodedSolution[i] ^= key[i];
					}
					callback({
						'correct': true,
						'solution': decoder.decode(encodedSolution),
					});
				});
			}
		} else {
			callback({ 'correct': false });
		}
	});
}

document.addEventListener('DOMContentLoaded', function() {
	const checkForm = document.getElementById('check');
	const checkInput = document.getElementById('check-input');
	const checkLabel = document.getElementById('check-label');
	const checkOut = document.getElementById('check-out');
	const checkSolution = document.getElementById('check-solution');

	const genForm = document.getElementById('gen');
	const genOuter = document.getElementById('gen-outer');
	const genOut = document.getElementById('gen-out');
	const genInput = document.getElementById('gen-input');
	const genSolution = document.getElementById('gen-solution');
	const genLink = document.getElementById('gen-link');

	// Always strings.
	let version = "";
	let salt = "";
	let hash = "";
	let label = "";
	let ciphertext = "";
	function updateFromHash() {
		const params = location.hash.substr(1).split('#');
		version = params[0] || "";
		salt = params[1] || "";
		hash = params[2] || "";
		label = params[3] || "";
		ciphertext = params[4] || "";
		if (version && salt && hash) {
			checkForm.style.display = "block";
			if (version === "0" && label) {
				checkLabel.textContent = "Check your answer for " + decodeURIComponent(label) + ":";
			} else if (version === "1" && label) {
				checkLabel.textContent = "Check your answer for " + decoder.decode(unb64(label)) + ":";
			} else {
				checkLabel.textContent = "Check your answer:";
			}
			genForm.style.display = "none";
			checkOut.style.display = 'none';
			checkSolution.style.display = 'none';
		} else {
			checkForm.style.display = "none";
			genForm.style.display = "block";
		}
	};

	checkForm.addEventListener('submit', function (event) {
		event.preventDefault();
		checkInput.select();
		const answer = canonicalize(checkInput.value);
		checkOut.style.display = 'block';
		checkOut.textContent = 'Checking...';
		checkOut.className = 'padded-callout';
		checkSolution.style.display = 'none';
		checkHash(version, label, salt, hash, answer, ciphertext, function (v) {
			if ('error' in v) {
				checkOut.className = 'padded-callout error';
				checkOut.textContent = 'Error: ' + v.error;
			} else if ('correct' in v) {
				checkOut.className = v.correct ? 'padded-callout success' : 'padded-callout error';
				checkOut.textContent = answer + ' is ' + (v.correct ? 'correct!' : 'incorrect.');
				if ('progress' in v) {
					checkOut.textContent += ' Decrypting (' + Math.floor(v.progress * 100) + '%)...';
				}
				if ('solution' in v) {
					checkSolution.style.display = 'block';
					checkSolution.className = 'padded-callout';
					checkSolution.textContent = v.solution;
				}
			} else if ('progress' in v) {
				checkOut.textContent = 'Checking (' + Math.floor(v.progress * 100) + '%)...';
			}
		});
	});

	updateFromHash();
	window.addEventListener('hashchange', updateFromHash);

	genForm.addEventListener('submit', function (event) {
		event.preventDefault();
		const answer = canonicalize(genInput.value);
		const solution = genSolution.value;
		genOuter.className = 'padded-callout';
		genOut.textContent = 'Generating...';
		genLink.textContent = '';
		genLink.href = '#';
		// this part is version 1 instead of version 0...
		const genLabel = b64OfArray(encoder.encode(document.getElementById('gen-label').value));
		generateHash(genLabel, answer, solution, function (v) {
			if ('error' in v) {
				genOuter.className = 'padded-callout error';
				genOut.textContent = 'Error: ' + v.error;
				genLink.textContent = '';
				genLink.href = '#';
			} else if ('version' in v && 'salt' in v && 'hash' in v) {
				genOuter.className = 'padded-callout success';
				genOut.textContent = 'Answer checker URL for ' + answer + ': ';
				let url = location.protocol + '//' + location.host + location.pathname + '#' + [v.version, v.salt, v.hash, genLabel].join('#');
				if ('ciphertext' in v) {
					url += '#' + v.ciphertext;
				}
				genLink.textContent = url;
				genLink.href = url;
			} else if ('progress' in v) {
				if (v.encrypting) {
					genOut.textContent = 'Encrypting (' + Math.floor(v.progress * 100) + '%)...';
				} else {
					genOut.textContent = 'Generating (' + Math.floor(v.progress * 100) + '%)...';
				}
				genLink.textContent = '';
				genLink.href = '#';
			}
		});
	});

	document.getElementById('loading').style.display = "none";
});
</script>

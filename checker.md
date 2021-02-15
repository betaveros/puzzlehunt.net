---
layout: page
short_title: Checker
title: Client-Side Answer Checker
---
A simple client-side answer checker for puzzlehunts. Of course, requires JavaScript. As is typical, considers every answer as a string of letters A–Z by changing lowercase letters to uppercase and stripping any other characters. Built on Richard Moore's [scrypt-js](https://github.com/ricmoo/scrypt-js), an implementation of Colin Percival's [scrypt](https://www.tarsnap.com/scrypt.html).

<p id="loading">Loading...</p>

<form id="check">
<p><label for="check-input"><strong id="check-label">Check your answer:</strong></label></p>
<p><input type="text" id="check-input" name="check-input" /> <button type="submit" id="check-submit">Check</button></p>
<p id="check-out"></p>
</form>

<form id="gen" style="margin-top: 4em;">
<p>To create an answer checker, enter your answer below and click Generate URL:</p>
<p>Correct answer: <input type="text" id="gen-input" /></p>
<p>What this answer checker is for (optional): <input type="text" id="gen-label" /></p>
<button type="submit">Generate URL</button>
<p id="gen-outer"><span id="gen-out"></span><a id="gen-link"></a></p>
</form>

### The Fine Print

This answer checker sort of tries to be cryptographically reasonable, but errs towards staying fast and not break, because this shouldn't be a high-security setting. Also no settings (see e.g. [this Beeminder essay](https://blog.beeminder.com/choices/), although URLs are versioned so we can keep things backwards-compatible). If you do have a use case for a client-side answer checker that requires stronger cryptographic guarantees, or (e.g.) a different policy for canonicalizing answers, let me know and I will consider it. Or just take this code and change it. There's really not that much.

The label is used in the hash in order to somewhat discourage a "meddler-in-the-middle" attack of the following form: You publish a puzzle using an answer checker URL, labeled to indicate your authorship and perhaps that the puzzle comes from a competitive setting. An adversary who can't solve your puzzle modifies your puzzle and the answer checker URL to remove your label and then republishes the modified puzzle and checker under their own name. Or, in the competitive setting, they can send the modified puzzle and checker URL to an innocent solver who is unaware of the puzzle's original context and trick them into solving it for the adversary. Including the label in the hash prevents these attacks, at the cost of making relabeling URLs harder for legitimate purposes. However, even with the label used in the hash, the adversary can rehost this checker or create a checker that proxies answer checks to it in a way that embeds the label used in the hash but doesn't surface it to the solver. Or they can just withhold the answer checker from the solver altogether and manually proxy any answers the solver wishes to check.

Does any of this matter? Probably not, but occupational hazard. You still probably shouldn't use this checker in any setting with actual stakes where competitive integrity is actually important. I mean, you can use this anywhere you think it fits, but don't sue me if somebody breaks the checker because I missed a different protocol break, or set the scrypt parameters incorrectly, or anything else. Let me copy in the liability incantation from the [Blue Oak Model License](https://blueoakcouncil.org/license/1.0.0): As far as the law allows, this software comes as is, without any warranty or condition, and no contributor will be liable to anyone for any damages related to this software or this license, under any kind of legal claim.

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

function generateHash(label, answer, callback) {
	const version = '0';
	// The caller should canonicalize the answer!
	const salt = generateLocalSalt();
	// Note: add the label even if it's empty. Also assume the label is ASCII
	// (by being URI-encoded) already.
	const fullSalt = encoder.encode("puzzlehunt.net/checker#" + version + '#' + salt + '#' + label);
	scrypt.scrypt(encoder.encode(answer), fullSalt, 4096, 8, 1, 24, function (progress) {
		callback({ 'progress': progress });
	}).then(function (key) {
		callback({
			'version': version,
			'salt': salt,
			'hash': b64OfArray(key),
		});
	});
}

function checkHash(version, label, salt, hash, answer, callback) {
	if (version !== '0') {
		callback({
			'error': 'Unsupported version: ' + version,
		});
	}
	// The caller should canonicalize the answer!
	// Note: add the label even if it's empty. Also assume the label is ASCII
	// (by being URI-encoded) already.
	const fullSalt = encoder.encode("puzzlehunt.net/checker#" + version + '#' + salt + '#' + label);
	scrypt.scrypt(encoder.encode(answer), fullSalt, 4096, 8, 1, 24, function (progress) {
		callback({ 'progress': progress });
	}).then(function (key) {
		if (b64OfArray(key) === hash) {
			callback({ 'correct': true });
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
	// Always strings.
	let version = "";
	let salt = "";
	let hash = "";
	let label = "";
	function updateFromHash() {
		const params = location.hash.substr(1).split('#');
		version = params[0] || "";
		salt = params[1] || "";
		hash = params[2] || "";
		label = params[3] || "";
		if (version && salt && hash) {
			checkForm.style.display = "block";
			if (label) {
				checkLabel.textContent = "Check your answer for " + decodeURIComponent(label) + ":";
			} else {
				checkLabel.textContent = "Check your answer:";
			}
		} else {
			checkForm.style.display = "none";
		}
	};

	checkForm.addEventListener('submit', function (event) {
		event.preventDefault();
		const answer = canonicalize(checkInput.value);
		checkOut.textContent = 'Checking...';
		checkOut.className = 'padded-callout';
		checkHash(version, label, salt, hash, answer, function (v) {
			if ('error' in v) {
				checkOut.className = 'padded-callout error';
				checkOut.textContent = 'Error: ' + v.error;
			} else if ('correct' in v) {
				checkOut.className = v.correct ? 'padded-callout success' : 'padded-callout error';
				checkOut.textContent = answer + ' is ' + (v.correct ? 'correct!' : 'incorrect.');
			} else if ('progress' in v) {
				checkOut.textContent = 'Checking (' + Math.floor(v.progress * 100) + '%)...';
			}
		});
	});

	updateFromHash();
	window.addEventListener('hashchange', updateFromHash);

	const genForm = document.getElementById('gen');
	const genOuter = document.getElementById('gen-outer');
	const genOut = document.getElementById('gen-out');
	const genInput = document.getElementById('gen-input');
	const genLink = document.getElementById('gen-link');

	genForm.addEventListener('submit', function (event) {
		event.preventDefault();
		const answer = canonicalize(genInput.value);
		genOuter.className = 'padded-callout';
		genOut.textContent = 'Generating...';
		genLink.textContent = '';
		genLink.href = '#';
		const genLabel = encodeURIComponent(document.getElementById('gen-label').value);
		generateHash(genLabel, answer, function (v) {
			if ('error' in v) {
				genOuter.className = 'padded-callout error';
				genOut.textContent = 'Error: ' + v.error;
				genLink.textContent = '';
				genLink.href = '#';
			} else if ('version' in v && 'salt' in v && 'hash' in v) {
				genOuter.className = 'padded-callout success';
				genOut.textContent = 'Answer checker URL for ' + answer + ': ';
				const url = location.protocol + '//' + location.host + location.pathname + '#' + [v.version, v.salt, v.hash, genLabel].join('#');
				genLink.textContent = url;
				genLink.href = url;
			} else if ('progress' in v) {
				genOut.textContent = 'Generating (' + Math.floor(v.progress * 100) + '%)...';
				genLink.textContent = '';
				genLink.href = '#';
			}
		});
	});

	genForm.style.display = "block";
	document.getElementById('loading').style.display = "none";
});
</script>

// 监听复制
document.addEventListener('copy', function(e){
e.clipboardData.setData('text/plain', 'Hello, world!');
e.clipboardData.setData('text/html', '<b>Hello, world!</b>');
e.preventDefault(); // We want our data, not data from any selection, to be written to the clipboard
});

// 手动触发复制
document.execCommand('copy')
